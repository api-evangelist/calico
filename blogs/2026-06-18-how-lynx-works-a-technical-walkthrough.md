---
title: "How Lynx Works: A Technical Walkthrough"
url: "https://www.tigera.io/blog/how-lynx-works-a-technical-walkthrough/"
date: "2026-06-18"
author: "Peter Kelly"
feed_url: "https://www.tigera.io/feed/"
---
A technical walkthrough of the architecture of Lynx, Tigera's new control plane for agentic AI traffic, providing a single control point in the path of every agent call with authentication, authorization, and recording. Its design requires no agent code modifications, adds no new databases to the control plane, and leverages the open-source agentgateway proxy, Kubernetes custom resources, SPIFFE/SPIRE or OIDC for workload identity, and Cedar policy language for authorization. Lynx also uses kernel-level eBPF monitoring to discover unregistered agents and applies RFC 8693 token exchange for delegation chains.
