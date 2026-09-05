---
name: calico-ip-pool-and-bgp
description: >-
  Add or change a Calico IP pool and configure BGP peering with the projectcalico.org/v3 API. Use
  this when asked to add pod address space, change encapsulation (IPIP/VXLAN), turn NAT-outgoing on
  or off, or peer Calico with a top-of-rack switch or route reflector.
api: Calico IPPool and BGPPeer APIs
base: https://{kube_apiserver_host}/apis/projectcalico.org/v3
operations:
  - listIPPool
  - createIPPool
  - readIPPool
  - replaceIPPool
  - deleteIPPool
  - listBGPPeer
  - createBGPPeer
  - readBGPPeer
  - replaceBGPPeer
  - deleteBGPPeer
  - listBGPConfiguration
  - readBGPConfiguration
---

# IP pools and BGP peering

## Read the world first

- `listIPPool` — what address space already exists, which pools are `disabled`, what `blockSize`
  they use, and their `nodeSelector`.
- `listBGPConfiguration` and `readBGPConfiguration` — the cluster-wide BGP settings. **These are
  read-only in this contract**: the OpenAPI declares no create/replace/delete for BGPConfiguration
  even though the CRD is writable. To change it, use `calicoctl apply` or `kubectl` against the CRD
  directly.
- `listBGPPeer` — existing sessions.

## Adding an IP pool

`createIPPool`. `spec.cidr` is the only required field, and everything else changes behaviour in
ways that are hard to see from the object:

- `blockSize` — how much of the CIDR each node claims at a time. Too large and small clusters
  exhaust the pool; too small and you churn allocations.
- `ipipMode` / `vxlanMode` — `Always`, `CrossSubnet`, or `Never`. Do not set both to `Always`.
  `CrossSubnet` encapsulates only when it has to, which is usually what you want.
- `natOutgoing` — whether pod traffic leaving the cluster is SNATed to the node IP.
- `nodeSelector` — which nodes may allocate from this pool. Selector-based, evaluated live.
- `allowedUses` — `Workload`, `Tunnel`, or both.
- `disabled: true` — the safe way to retire a pool. It stops NEW allocations while leaving existing
  ones alone.

**Never overlap a new pool's CIDR with an existing pool or with any real network the cluster routes
to.** Nothing in the API will stop you, and the failure shows up as unreachable pods, not as an
error on the write.

## Retiring a pool

Prefer `replaceIPPool` with `spec.disabled: true` over `deleteIPPool`. Disabling is a state you can
set back; deleting is not — there is no reversal operation and no window
(`conventions/calico-conventions.yml`). Workloads already holding addresses from a deleted pool keep
them, which makes the damage slow and confusing rather than immediate and obvious.

## BGP peering

`createBGPPeer`. Choose the pairing deliberately:

- `node` + `peerIP` + `asNumber` — one named node peers with one specific address. Explicit.
- `nodeSelector` + `peerIP` — every node matching the labels peers with that address. This is the
  top-of-rack pattern.
- `peerSelector` — peer with other Calico nodes matching a selector. This is the route-reflector
  pattern.

Other fields that matter: `sourceAddress` (`UseNodeIP` or `None`), `keepOriginalNextHop`,
`ttlSecurity` (GTSM hop limit), `maxRestartTime` (graceful restart).

### The password field crosses out of Calico

`spec.password.secretKeyRef` points at a **core/v1 Secret** — `{name, key}` — not at anything in the
`projectcalico.org` group. Two consequences:

1. The caller needs RBAC on Secrets in that namespace, not just on `projectcalico.org`. A 403 here
   is about the Secret, not about the BGPPeer.
2. Create the Secret before the BGPPeer. A peer referencing a missing Secret will not come up, and
   the BGPPeer object itself will look perfectly healthy.

## Writing safely

`replaceIPPool` and `replaceBGPPeer` are full replacements. Read, keep `metadata.resourceVersion`,
modify, send it back. A 409 means re-read and retry the change against the fresh copy — never blank
`resourceVersion`.

`createIPPool` and `createBGPPeer` are **not** replay-safe: a repeat returns `409 AlreadyExists`.
Prefer `calicoctl apply`, which creates-if-absent and replaces-if-present, when you need a write
you can run twice.

## Blast radius

`deleteBGPPeer` tears down the session and withdraws every route learned over it. On a cluster that
depends on BGP for pod reachability, that is a full outage for the affected paths, and it takes
effect immediately.
