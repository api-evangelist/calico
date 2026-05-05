---
title: "Secure and Scale VMware VKS with Calico Kubernetes Networking"
url: "https://www.tigera.io/blog/vmware-vks-calico-secure-networking/"
date: "Sun, 22 Mar 2026 18:50:39 +0000"
author: "Abhishek Rao"
feed_url: "https://www.tigera.io/feed/"
---
<div style="border-left: 3px solid #00a3e0; padding-left: 15px; margin: 25px 0; line-height: 1.6;">
<p style="margin: 0; color: #666; font-size: 14px; letter-spacing: 1px;">Co-authors</p>
<p style="margin: 0; color: #333; font-size: 16px;"><strong>Abhishek Rao</strong> <span style="color: #00a3e0;">|</span> Tigera<br />
<strong>Ka Kit Wong, Charles Lee, &amp; Christian Rauber</strong> <span style="color: #00a3e0;">|</span> Broadcom</p>
</div>
<p><a href="https://www.vmware.com/products/cloud-infrastructure/vsphere-kubernetes-service" rel="noopener" target="_blank">VMware vSphere Kubernetes Service (VKS)</a> is the CNCF-certified Kubernetes runtime built directly into VMware Cloud Foundation (VCF), which delivers a single platform for both virtual machines and containers. VKS enables platform engineers to deploy, manage, and scale Kubernetes clusters while leveraging a comprehensive set of cloud services. And with VKS v3.6, that foundation just got significantly more powerful: VKS now natively supports Calico Enterprise — part of the <a href="https://www.tigera.io/tigera-products/calico-commercial-editions/" rel="noopener" target="_blank">Calico Unified Platform</a> — as a validated, lifecycle-managed networking add-on through the new VKS Addon Framework. This integration is a key milestone in <a href="https://blogs.vmware.com/cloud-foundation/2026/03/23/accelerating-customer-success-with-expanded-partnerships-across-the-kubernetes-ecosystem/" rel="noopener" target="_blank">VMware’s expanded partnerships across the Kubernetes ecosystem</a>, ensuring customers have access to best-in-class networking and security tools.</p>
<p>Even better, VKS natively integrates <a href="https://www.tigera.io/tigera-products/calico/" rel="noopener" target="_blank">Calico Open Source</a> by Tigera as a supported, out-of-the-box Container Network Interface (CNI). This gives organizations a powerful open source baseline right from day one:</p>
<ul>
<li><strong>Pluggable Data Planes:</strong> The flexibility to run high-performance eBPF, standard Linux iptables, modern nftables, or Windows data planes based on specific workload needs.</li>
<li><strong>Wire-Speed Routing:</strong> Direct BGP peering with the underlying VMware NSX infrastructure, eliminating the performance overhead of traditional overlay networks.</li>
<li><strong>Foundational Zero-Trust:</strong> Global default-deny policies to instantly secure pod-to-pod traffic.</li>
<li><strong>Observability:</strong> Includes Whisker, a visual UI tool that simplifies access to flow logs, making it easier to analyze network communication and debug policies.</li>
</ul>
<p>VKS and Calico Open Source build the perfect house for your applications. However, as Kubernetes adoption explodes across the enterprise, platform engineering and security teams inevitably hit a new wall.</p>
<p>What happens when your security team mandates strict compliance audits across 50 different clusters? What happens when you need to route ephemeral Kubernetes traffic through your legacy physical firewalls? Or when a critical microservice drops traffic at 2 AM and you need to know exactly why?</p>
<p>To conquer the complex realities of production scale, organizations running VKS are supercharging their environments with the <a href="https://www.tigera.io/tigera-products/calico-commercial-editions/" rel="noopener" target="_blank">Calico Unified Platform</a> (available via Calico Enterprise and Calico Cloud). Here is how Calico transforms your baseline VKS clusters into a fully observable, enterprise-grade networking and security platform.</p>
<hr />
<h3>The Calico Unified Platform Reference Architecture</h3>
<p>As you scale your VKS environment, your architecture must evolve from providing basic pod connectivity to delivering a comprehensive security, routing, and observability mesh.</p>
<p>The reference architecture below illustrates how Calico Unified Platform wraps your VKS worker nodes in advanced Layer 7 protections, granular egress controls, and deep forensic logging capabilities—all while maintaining the high-performance eBPF and BGP foundation of your clusters.</p>
<p><strong>Calico Unified Platform Architecture</strong></p>
<p><a href="https://wordpress-1075849-4005834.cloudwaysapps.com/app/uploads/2026/03/image1.png"><img alt="" class="alignnone size-full wp-image-67529" height="1251" src="https://wordpress-1075849-4005834.cloudwaysapps.com/app/uploads/2026/03/image1.png" width="1999" /></a></p>
<p><em>Figure 1: Calico Unified Platform reference architecture for VKS &#8211; showing how Calico Enterprise wraps VKS worker nodes with Layer 7 security, egress controls, and deep observability while preserving the eBPF and BGP performance foundation.</em></p>
<hr />
<h3>1. Secure the Perimeter: Bridging Kubernetes with Legacy Firewalls</h3>
<p>Traditional network security teams often struggle with Kubernetes because Pod IP addresses are ephemeral—they spin up and die in seconds. This makes it virtually impossible to write static firewall rules on your external Palo Alto or Fortinet appliances.</p>
<p>The Calico Unified Platform bridges this gap seamlessly for VKS environments:</p>
<ul>
<li><strong>Egress Gateway &amp; Source NAT:</strong> Calico allows you to map dynamic Kubernetes namespaces to highly available, static IP Egress Gateways. When a pod talks to the outside world, your external firewall only sees the static IP. No more fighting with the NetSec team over IP tracking!</li>
<li><strong>Native WAF and IDS/IPS:</strong> Secure your inbound traffic right at the Calico Ingress Gateway. Calico integrates a powerful Web Application Firewall (WAF) using the ModSecurity Core Rule Set. Coupled with native Intrusion Detection/Prevention (IDS/IPS) and DDoS protection, Calico detects and blocks malicious payloads before they impact performance.</li>
<li><strong>DNS Policies &amp; Threat Feeds:</strong> Do not just block IPs; block malicious domains. Calico dynamically ingests global threat intelligence feeds to automatically halt traffic to known bad actors.</li>
</ul>
<h3>2. Enforce Zero-Trust at Scale: Unified Policy Across Kubernetes, VMs, and Bare Metal</h3>
<p>Open-source network policies are fantastic, but managing them across dozens of teams and clusters can quickly turn into the “Wild West” of YAML files. Calico brings true enterprise governance to your VKS environment—and extends it well beyond Kubernetes:</p>
<ul>
<li><strong>Network Policy Tiers &amp; Staged Policies:</strong> A hierarchical, RBAC-driven approach to security. The Security team can create non-overrideable “Tier 1” guardrails, while Developers get full freedom to write microsegmentation rules for their specific namespaces. Even better, with Staged Policies, you can preview and test the impact of any rule on live traffic before fully enforcing it, ensuring zero downtime.</li>
<li><strong>Unified Protection for Legacy VMs &amp; Bare Metal:</strong> Your VKS clusters do not exist in a vacuum. Calico extends its policy engine beyond Kubernetes, allowing you to secure traditional VMware VMs and bare-metal servers using the exact same single-pane-of-glass dashboard—a headline differentiator of the Calico Unified Platform.</li>
<li><strong>Sidecar-Less Service Mesh (Istio Ambient Mode):</strong> Get the deep L7 visibility and mTLS encryption of a service mesh without the crippling performance overhead. Calico seamlessly integrates with Istio Ambient Mesh, managed through a single Calico operator—no standalone Istio expertise required.</li>
</ul>
<h3>3. Total Visibility: One Management Plane for Every Traffic Flow</h3>
<p>When a connection fails in a standard K8s cluster, troubleshooting usually involves blindly digging through kubectl logs. It is slow, frustrating, and drastically inflates your Mean Time to Resolution (MTTR).</p>
<p>Calico acts as the ultimate CCTV system for your VKS clusters—with a single console covering every traffic type, from ingress to egress to pod-to-pod:</p>
<ul>
<li><strong>Dynamic Service Graph &amp; Alerts:</strong> Get a real-time visual map of all microservice traffic across your clusters. Instantly see performance metrics, blocked traffic, and active connections. You can even configure automated alerts and incident response to deploy mitigating policies the second an anomaly is detected.</li>
<li><strong>Deep Forensic Logging:</strong> Calico goes far beyond basic flow logs. It provides granular DNS Logs, L7 Logs, and Ingress Logs, allowing you to pinpoint exactly which layer of the stack is failing.</li>
<li><strong>On-Demand Packet Capture:</strong> Did a specific pod trigger an anomaly? Trigger a targeted packet capture (pcap) directly from the Calico UI for deep forensic analysis, without ever having to SSH into the vSphere worker nodes.</li>
</ul>
<h3>4. Scale Without Limits: Multi-Cluster Management and AI-Powered Operations</h3>
<p>As your VMware footprint grows, managing clusters individually becomes impossible. Calico’s Multi-Cluster Management provides a single pane of glass to view, secure, and troubleshoot all your VKS clusters—and even your public cloud EKS/AKS clusters. You can seamlessly federate identities and extend resilient multi-cluster networking with Cluster Mesh.</p>
<p>And when things get truly complex? AI Assistant for Calico serves as your platform co-pilot. You can use natural language prompts to generate declarative Policy as Code, query flow logs, and diagnose active threats, drastically reducing the learning curve for new team members.</p>
<h3>The Ultimate VKS Experience</h3>
<p>VMware VKS gives you a world-class, CNCF-certified Kubernetes platform built directly into VCF. Calico Enterprise — part of the <a href="https://www.tigera.io/tigera-products/calico-commercial-editions/" rel="noopener" target="_blank">Calico Unified Platform</a> — takes that foundation further, delivering a single management plane for networking, network security, and observability across every cluster, every workload type, and every environment. No stitching tools together. No integration tax. Just the enterprise-grade performance and security your most critical workloads demand.</p>
<div style="background-color: #f0f7f9; border-left: 5px solid #00a3e0; padding: 25px; margin: 30px 0; border-radius: 4px;">
<h4 style="margin-top: 0; color: #002f35;">Ready to see it in action?</h4>
<p style="margin-bottom: 10px;"><a href="https://www.tigera.io/demo/" style="color: #00a3e0; font-weight: bold; text-decoration: none;">Request a Demo of Calico Enterprise →</a></p>
<p style="margin-bottom: 0;"><a href="https://www.calicocloud.io/home" style="color: #00a3e0; font-weight: bold; text-decoration: none;">Start your free trial of Calico Cloud today →</a></p>
</div>
<p>The post <a href="https://www.tigera.io/blog/vmware-vks-calico-secure-networking/">Secure and Scale VMware VKS with Calico Kubernetes Networking</a> appeared first on <a href="https://www.tigera.io">Tigera – Creator of Calico</a>.</p>
