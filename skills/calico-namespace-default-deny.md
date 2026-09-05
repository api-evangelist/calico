---
name: calico-namespace-default-deny
description: >-
  Lock a Kubernetes namespace to default-deny with Calico and then open exactly the traffic the
  workloads need, using the projectcalico.org/v3 NetworkPolicy API. Use this when asked to isolate a
  namespace, apply zero-trust to a team's workloads, or stop a namespace talking to everything.
api: Calico NetworkPolicy API
base: https://{kube_apiserver_host}/apis/projectcalico.org/v3
operations:
  - listNamespacedNetworkPolicy
  - createNamespacedNetworkPolicy
  - readNamespacedNetworkPolicy
  - replaceNamespacedNetworkPolicy
  - deleteNamespacedNetworkPolicy
---

# Lock a namespace to default-deny

## Before you touch anything

**There is no undo.** Calico publishes no reversal operation and no reversal window for any write in
this API (`conventions/calico-conventions.yml`). A default-deny policy applied to a live namespace
stops traffic the instant the API server accepts it. Rehearse first.

1. `listNamespacedNetworkPolicy` on the target namespace. Read what is already there — `spec.order`
   and `spec.tier` decide evaluation order, and a policy you did not know about may already be
   allowing or denying the traffic you are about to reason about.
2. Write the policy as a **StagedNetworkPolicy** first. Staged policies are evaluated and reported
   but never enforced. This is the only real rehearsal Calico gives you for policy.
3. Or, at minimum, send the create with `--dry-run=server` so admission and CRD schema validation
   run without persisting.

## The default-deny policy

`createNamespacedNetworkPolicy` in the target namespace with a policy that selects everything and
declares both directions with no rules:

- `spec.selector: all()` — matches every endpoint in the namespace.
- `spec.types: [Ingress, Egress]` — declaring a type with no matching rule is what makes it deny.
- `spec.order` — a low number evaluates earlier. Pick deliberately; do not leave it unset and hope.

Omitting `types` is the classic mistake: a policy with no `types` and no rules denies nothing.

## Open what the workloads actually need

For each required flow, `createNamespacedNetworkPolicy` with an `order` lower than the deny policy
(lower evaluates first) and a `spec.ingress[]` or `spec.egress[]` entry whose `action` is `Allow`.

Inside a `Rule`, `source` and `destination` are `EntityRule` objects. Reach for, in order of
preference:

- `selector` / `namespaceSelector` — label-based, and the whole point of Calico. Membership is
  re-evaluated as pods come and go.
- `serviceAccounts` — identity-based rather than location-based.
- `nets` — CIDRs, for things outside the cluster.

Almost every namespace needs egress to DNS (`kube-system`, UDP/TCP 53) before anything else works.
Add it deliberately; a namespace that cannot resolve names fails in ways that look nothing like a
network policy problem.

## Changing a policy later

`replaceNamespacedNetworkPolicy` is a full replacement, not a merge.

1. `readNamespacedNetworkPolicy` and keep `metadata.resourceVersion`.
2. Modify the object you read.
3. `replaceNamespacedNetworkPolicy` sending that same `resourceVersion`.

A `409 Conflict` means someone wrote first. Re-read, re-apply your change to the fresh copy, retry.
**Never blank `resourceVersion` to force the write through** — that converts a safe conflict into a
silent overwrite of another operator's change (`errors/calico-problem-types.yml`).

## Errors you will actually meet

| Status | reason | What to do |
|---|---|---|
| 401 | Unauthorized | No valid bearer token or client cert. |
| 403 | Forbidden | RBAC does not grant the verb on `networkpolicies` in `projectcalico.org`. **Not declared in the spec** — expect it anyway. |
| 404 | NotFound | Wrong name or namespace — or Calico's CRDs/aggregation API server are not installed. Check that before assuming the object is missing. |
| 409 | AlreadyExists / Conflict | On create, it exists. On replace, `resourceVersion` is stale. |
| 422 | Invalid | Schema or admission validation failed. Read `status.details.causes[]`. **Not declared in the spec.** |

Errors are the Kubernetes `metav1.Status` envelope, not RFC 9457. Branch on `reason`, never on
`message`.

## Deleting

`deleteNamespacedNetworkPolicy` is immediate and irreversible. In a default-deny namespace, deleting
the policy that *permits* traffic breaks the workloads; deleting the policy that *denies* traffic
silently opens them. Keep the manifest in version control — re-applying it is the only reversal
that exists, and it is yours, not Calico's.
