# Calico (calico)

Calico is an open source networking and network security solution for containers, virtual machines, and native host-based workloads. Created and maintained by Tigera, it is the most widely adopted solution for container networking and security, powering over 8 million nodes daily across 166 countries.

**APIs.json:** [https://raw.githubusercontent.com/api-evangelist/calico/refs/heads/main/apis.yml](https://raw.githubusercontent.com/api-evangelist/calico/refs/heads/main/apis.yml)

## Scope

- **Type:** Index

## Tags

- CNI
- Containers
- eBPF
- Kubernetes
- Network Policy
- Network Security
- Networking
- Open Source
- Service Mesh

## Timestamps

- **Created:** 2026-03-26
- **Modified:** 2026-04-23

## APIs

### Calico Client API

The Calico Client Library provides programmatic access to manage Calico resources such as network policies, IP pools, BGP configuration, host and workload endpoints, and IPAM settings. It is the core programmatic interface consumed by calicoctl and other Calico components for managing container networking and security resources.

- **Human URL:** [https://docs.tigera.io/calico/latest/reference/](https://docs.tigera.io/calico/latest/reference/)

#### Tags

- Client Library
- CNI
- Network Policy
- Networking

#### Properties

- [Documentation](https://docs.tigera.io/calico/latest/reference/)
- [Git Hub](https://github.com/projectcalico/calico)
- [Postman Collection](collections/calico.postman_collection.json) — [Postman Collection 2.1](https://schema.getpostman.com/json/collection/v2.1.0/collection.json)
- [Open Collection](collections/calico.opencollection.json) — [Open Collection 1.0](https://schema.opencollection.com/opencollection/v1.0.0.json)

### calicoctl CLI

calicoctl is the command-line tool that enables operators and automation systems to create, read, update, and delete Calico resources such as policies, IP pools, BGP peers, host endpoints, and workload endpoints. It also supports datastore migration, IPAM management, node diagnostics, and cluster status operations.

- **Human URL:** [https://docs.tigera.io/calico/latest/reference/calicoctl/](https://docs.tigera.io/calico/latest/reference/calicoctl/)

#### Tags

- CLI
- IPAM
- Network Policy
- Operations

#### Properties

- [Documentation](https://docs.tigera.io/calico/latest/reference/calicoctl/)
- [Git Hub](https://github.com/projectcalico/calicoctl)
- [Postman Collection](collections/calico.postman_collection.json) — [Postman Collection 2.1](https://schema.getpostman.com/json/collection/v2.1.0/collection.json)
- [Open Collection](collections/calico.opencollection.json) — [Open Collection 1.0](https://schema.opencollection.com/opencollection/v1.0.0.json)

### Calico Kubernetes CRDs

Calico exposes its networking and security primitives through Kubernetes Custom Resource Definitions (CRDs) including NetworkPolicy, GlobalNetworkPolicy, IPPool, BGPConfiguration, BGPPeer, HostEndpoint, WorkloadEndpoint, FelixConfiguration, and others. These CRDs allow declarative management of container networking and security via the Kubernetes API.

- **Human URL:** [https://docs.tigera.io/calico/latest/reference/resources/](https://docs.tigera.io/calico/latest/reference/resources/)

#### Tags

- CRDs
- Kubernetes
- Network Policy
- Security

#### Properties

- [Documentation](https://docs.tigera.io/calico/latest/reference/resources/)
- [Git Hub](https://github.com/projectcalico/calico)
- [Postman Collection](collections/calico.postman_collection.json) — [Postman Collection 2.1](https://schema.getpostman.com/json/collection/v2.1.0/collection.json)
- [Open Collection](collections/calico.opencollection.json) — [Open Collection 1.0](https://schema.opencollection.com/opencollection/v1.0.0.json)

## Common Properties

- [LinkedIn](https://www.linkedin.com/company/calico-life-sciences-llc)
- [Website](https://www.tigera.io/project-calico/)
- [Documentation](https://docs.tigera.io/)
- [Getting Started](https://docs.tigera.io/calico/latest/getting-started/kubernetes/quickstart)
- [GitHub Organization](https://github.com/projectcalico)
- [GitHub Repository](https://github.com/projectcalico/calico)
- [Blog](https://www.tigera.io/blog/)
- [Pricing](https://www.tigera.io/tigera-products/calico/)
- [Slack](https://slack.projectcalico.org/)
- [Training](https://www.tigera.io/interactive-training/)
- [Certification](https://www.tigera.io/lp/calico-certification/)
- [Integrations](https://www.tigera.io/partners/)

## Maintainers

**FN:** Kin Lane
**Email:** kin@apievangelist.com
