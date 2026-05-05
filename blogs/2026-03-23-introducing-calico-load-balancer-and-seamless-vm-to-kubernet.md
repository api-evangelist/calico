---
title: "Introducing Calico Load Balancer and Seamless VM-to-Kubernetes Migration"
url: "https://www.tigera.io/blog/tigera-news-kubecon-amsterdam-2026/"
date: "Mon, 23 Mar 2026 07:01:36 +0000"
author: "Tigera Team"
feed_url: "https://www.tigera.io/feed/"
---
<p><strong>SAN JOSE, Calif., March 23, 2026</strong> — <a href="https://www.tigera.io/?utm_source=syndicate&amp;utm_medium=press_release&amp;utm_campaign=KubeCon2026" rel="noopener" target="_blank">Tigera</a>, the creator and maintainer of Project Calico, today announced a major expansion of its Unified Network Security Platform for Kubernetes, aimed at helping enterprises consolidate infrastructure and accelerate the migration of legacy workloads to cloud-native platforms.</p>
<p>The new capabilities include:</p>
<ul>
<li><strong><a href="#load-balancer">Calico Load Balancer</a>:</strong> A high-performance, eBPF-based, software-defined load balancer that replaces expensive, rigid hardware appliances with a Kubernetes-native solution.</li>
<li><strong><a href="#vm-migration">Seamless VM-to-Kubernetes Migration</a>:</strong> Advanced Layer 2 (L2) networking support eliminates migration friction by allowing virtual machines to move into Kubernetes clusters without changing their original IP addresses or existing VLAN dependencies.</li>
</ul>
<p>These innovations help organizations tackle the rising &#8220;complexity tax&#8221; in managing high-scale Kubernetes clusters and provide a high-velocity path to consolidate virtual machines and containers into a single, standardized platform.</p>
<div style="background-color: #dceaf3; padding: 25px; margin: 30px 0; color: #041a57;">
<p style="font-style: italic; font-size: 1.15em; line-height: 1.5; margin-bottom: 15px;">&#8220;The industry is at a breaking point where the operational overhead of managing legacy hardware and fragmented VM silos is no longer sustainable. By building a distributed load balancer into the fabric of Calico, and introducing live migration support to move VMs to Kubernetes, we are giving platform teams the power to innovate rather than spend hours managing and troubleshooting.&#8221;</p>
<p style="font-weight: bold; margin-top: 10px; font-size: 0.9em; letter-spacing: 1px;">— Ratan Tipirneni, president and CEO, Tigera</p>
</div>
<h2 id="load-balancer">Eliminating Hardware Bottlenecks: The Calico Load Balancer</h2>
<p>On-premises Kubernetes teams have traditionally relied on legacy hardware appliances to expose services, creating significant operational overhead and rigid dependencies between networking and platform teams. These external solutions often lack visibility into Kubernetes service context, do not scale horizontally, and require manual coordination for even basic software upgrades.</p>
<p>Tigera is disrupting this model with the Calico Load Balancer, a modern, software-defined solution built natively into the Calico platform. By transforming existing cluster nodes into a distributed, session-stable load-balancing tier, platform teams gain full control over service advertisement and configuration using the same Kubernetes workflows they already use.</p>
<p>This Kubernetes-native innovation delivers several critical advantages:</p>
<ul>
<li><strong>Session Persistence for Stateful Apps:</strong> A high-performance, eBPF-based data plane ensures that latency-sensitive, stateful applications like Kafka or RabbitMQ maintain active connections even during node failures or changes in network paths.</li>
<li><strong>Graceful Node Restarts:</strong> Platform teams can mark nodes for maintenance and take them offline without impacting user sessions, preventing lost transactions for critical business services.</li>
<li><strong>Reduced Latency:</strong> By enabling return traffic to take a shorter path back to the client, the solution reduces latency compared to traditional appliances where traffic must pass through the same central hardware twice.</li>
<li><strong>Simplified Scaling:</strong> The load balancer scales horizontally with the cluster; adding more nodes automatically adds more load-balancing capacity without vertical scaling limits or vendor upgrade cycles.</li>
<li><strong>Self-Service and Declarative Control:</strong> Configuration is handled through standard Kubernetes resources and GitOps workflows, removing cross-team bottlenecks and eliminating the need for tickets or separate management consoles.</li>
</ul>
<p><strong><em>Technical Deep Dive: <a href="https://www.tigera.io/blog/calico-load-balancer-simplifying-network-traffic-management-with-ebpf/">Simplifying network traffic management with eBPF and the Calico Load Balancer.</a></em></strong></p>
<h2 id="vm-migration">The Great Migration: Seamlessly Moving VMs to Kubernetes</h2>
<p>Historically, migrating virtual machines to Kubernetes meant a forced network redesign because VMs rely on static IP addresses and legacy Layer 2 VLAN configurations. Tigera&#8217;s new L2 networking support removes this friction.</p>
<ul>
<li><strong>Zero-Change Migration:</strong> VMs can be migrated from VMware to Kubernetes (KubeVirt) while keeping their original IP addresses, ensuring business continuity for applications with hardcoded dependencies.</li>
<li><strong>Instant Security Upgrade:</strong> Once migrated, VMs are automatically protected by Calico&#8217;s microsegmentation, allowing organizations to retire costly third-party security tools.</li>
</ul>
<p>Once migrated, the VMs in Kubernetes benefit from Calico&#8217;s advanced network security and observability capabilities. For users familiar with technologies like VMware NSX, Calico provides NSX-like functionality, including software-defined networking, microsegmentation, a workload-based firewall, and egress gateways for VMs running in Kubernetes.</p>
<p><strong><em>Step-by-Step Guide: <a href="https://www.tigera.io/blog/lift-and-shift-vms-to-kubernetes-with-calico-l2-bridge-networks/">Lift and shift VMs to Kubernetes with Calico L2 bridge networks.</a></em></strong></p>
<h2>One Platform for Networking, Security, and Observability</h2>
<p>The new Calico Unified Network Security Platform provides platform teams with a single, operator-managed solution. This allows teams to gain consistent network policy enforcement across L3-L7 layers with unified visibility, eliminating the overhead of managing multiple tools. Calico works consistently across any Kubernetes distribution, virtual machines, and bare-metal servers, ensuring enterprises can avoid vendor lock-in.</p>
<p><strong>About Tigera</strong></p>
<p><a href="https://www.tigera.io/?utm_source=syndicate&amp;utm_medium=press_release&amp;utm_campaign=KubeCon2026" rel="noopener" target="_blank">Tigera</a> provides Calico, a unified network security and observability platform to prevent, detect, and mitigate security breaches in Kubernetes clusters. Tigera&#8217;s open-source offering, <a href="https://www.tigera.io/tigera-products/calico?utm_source=syndicate&amp;utm_medium=press_release&amp;utm_campaign=KubeCon2026" rel="noopener" target="_blank">Calico Open Source</a>, is the most widely adopted container networking and security solution. Powering more than 100M containers across 8M+ nodes, Calico is supported across all major cloud providers and Kubernetes distributions.</p>
<p>Media Contact<br />
Media relations, Tigera<br />
contact@tigera.io</p>
<hr style="border: 0; height: 1px; background: #e0e0e0; margin: 40px 0;" />
<div style="background-color: #f9f9f9; padding: 30px; margin: 40px 0;">
<h3 style="margin-top: 0; color: #09287d;">Next Steps: Get Hands-on with These Innovations</h3>
<p style="font-size: 1.1em; line-height: 1.6;">Learn more about Calico Load Balancer and L2 networking support within the Calico ecosystem. Whether you are looking to optimize troubleshooting, reduce hardware dependency, or accelerate your VM migration, we provide the tools to get started today.</p>
<ul style="padding-left: 0; margin-top: 20px;">
<li style="margin-bottom: 12px;"><img alt="🚀" class="wp-smiley" src="https://s.w.org/images/core/emoji/17.0.2/72x72/1f680.png" style="height: 1em;" /> <strong>Experience the Platform:</strong> <a href="https://www.calicocloud.io/" rel="noopener" style="color: #09287d; font-weight: bold; text-decoration: underline;" target="_blank">Start a free trial of Calico Cloud</a></li>
<li><img alt="📅" class="wp-smiley" src="https://s.w.org/images/core/emoji/17.0.2/72x72/1f4c5.png" style="height: 1em;" /> <strong>Personalized Deep Dive:</strong> <a href="https://www.tigera.io/demo/" rel="noopener" style="color: #09287d; font-weight: bold; text-decoration: underline;" target="_blank">Request a technical demo with our engineering team</a></li>
</ul>
<p style="margin-top: 20px; font-size: 0.95em; color: #555;"><em>Attending KubeCon Amsterdam Mar 23-26, 2026? Stop by the Tigera booth #400 to learn more about these features.</em></p>
</div>
<hr style="border: 0; height: 1px; background: #e0e0e0; margin: 40px 0;" />
<p>The post <a href="https://www.tigera.io/blog/tigera-news-kubecon-amsterdam-2026/">Introducing Calico Load Balancer and Seamless VM-to-Kubernetes Migration</a> appeared first on <a href="https://www.tigera.io">Tigera – Creator of Calico</a>.</p>
