---
title: "KubeVirt Networking: How to Preserve VM IP Addresses During Migration"
url: "https://www.tigera.io/blog/kubevirt-networking-how-to-preserve-vm-ip-addresses-during-migration/"
date: "Tue, 21 Apr 2026 20:55:58 +0000"
author: "Dillon Barry"
feed_url: "https://www.tigera.io/feed/"
---
<p>Organisations are re-evaluating their VM infrastructure. The economics have shifted, the tooling has matured, and the case for running two separate platforms, one for containers, one for VMs, is getting harder to justify. Platform teams that spent years managing hypervisor infrastructure are being asked to consolidate, and most are landing on the same answer: Kubernetes.</p>
<p><a href="https://www.tigera.io/learn/guides/kubevirt/">KubeVirt</a> makes running VMs on Kubernetes possible. But <a href="https://www.tigera.io/blog/deep-dive/the-power-of-kubevirt-and-calico/">KubeVirt networking</a> &#8211; what happens to a VM&#8217;s IP address, VLAN, and security posture when it lands in a cluster &#8211; is where most migration plans hit a wall. The reasons go beyond cost:</p>
<ul>
<li><strong>Most enterprises already run Kubernetes.</strong> Containers are already there. Adding VMs to the same platform consolidates tooling, lifecycle management, networking models, and security policy into a single operational model.</li>
<li><strong>Two platforms means double the overhead.</strong> Separate infrastructure means separate upgrade cycles, separate monitoring, separate network configuration, and separate on-call runbooks. Platform consolidation has direct operational value.</li>
<li><strong>Kubernetes is mature enough.</strong> KubeVirt has reached the point where it’s a viable production choice for enterprise VM workloads.</li>
</ul>
<p>The decision to migrate is being made. The question is <strong>how to do it without causing chaos.</strong></p>
<h2>Introducing KubeVirt</h2>
<p><a href="https://kubevirt.io/">KubeVirt</a> extends the Kubernetes API with new custom resource types: <code>VirtualMachine</code> and <code>VirtualMachineInstance</code>. These make VMs first-class Kubernetes objects — scheduled, managed, and observable through the same tools and APIs as containers.</p>
<p>A VM running in KubeVirt runs inside a <code>virt-launcher pod</code>. Kubernetes schedules that pod to a node with available resources, the same way it schedules any other workload. The VM gets CPU and memory from the node. It doesn&#8217;t know it moved.</p>
<p>That’s the point: from the VM’s perspective, KubeVirt is invisible. The operating system keeps running. The application keeps running.</p>
<figure class="wp-caption aligncenter" id="attachment_67953" style="width: 676px;"><img alt="" class="size-full wp-image-67953" height="254" src="https://www.tigera.io/app/uploads/2026/04/KubeVirt-Networking-How-to-Preserve-VM-IP-Addresses-During-Migration-1.png" width="676" /><figcaption class="wp-caption-text" id="caption-attachment-67953">KubeVirt virt-launcher pods in Kubernetes</figcaption></figure>
<h2>The network is a different story</h2>
<p>When you migrate a VM, three things have to follow: compute, storage, and network. Compute and storage are properties of the VM itself — self-contained. KubeVirt handles them by giving the VM a new host and a new storage backend. The VM doesn’t notice.</p>
<table style="border-collapse: collapse; width: 100%;">
<tbody>
<tr>
<td style="width: 33.3333%;"><strong>Dependency</strong></td>
<td style="width: 33.3333%;"><strong>What KubeVirt Does</strong></td>
<td style="width: 33.3333%;"><strong>Status</strong></td>
</tr>
<tr>
<td style="width: 33.3333%;"><strong>Compute </strong></td>
<td style="width: 33.3333%;">VM runs in a virt-launcher pod. Kubernetes schedules it.</td>
<td style="width: 33.3333%;">Solved</td>
</tr>
<tr>
<td style="width: 33.3333%;"><strong>Storage </strong></td>
<td style="width: 33.3333%;">Disk images mapped to Persistent Volumes via migration tools.</td>
<td style="width: 33.3333%;">Solved</td>
</tr>
<tr>
<td style="width: 33.3333%;"><strong>Network</strong></td>
<td style="width: 33.3333%;">VM gets a new IP from the Kubernetes pod CIDR.</td>
<td style="width: 33.3333%;">Note Solved</td>
</tr>
</tbody>
</table>
<p style="text-align: center;"><em>Dependencies of VM migrations</em></p>
<p>The network is different. The network isn’t a property of the VM. <strong>It’s a property of the VM’s relationship to everything else in the infrastructure.</strong></p>
<p>A VM’s compute dependency is between the VM and its host. A VM’s storage dependency is between the VM and a storage backend. But a VM’s network dependency is between the VM and every other system that knows how to reach it.</p>
<p>That distinction is why networking is where VM migrations stall. This isn&#8217;t theoretical. KubeVirt&#8217;s own issue tracker documents the problem directly: a user <a href="https://github.com/kubevirt/kubevirt/issues/14320">reported their VM&#8217;s IP changing after live migration</a>, and a project maintainer confirmed: &#8220;Sticky IPs is not implemented.&#8221; The network identity doesn&#8217;t follow the VM by default.</p>
<figure class="wp-caption aligncenter" id="attachment_67954" style="width: 1164px;"><img alt="" class="size-full wp-image-67954" height="850" src="https://www.tigera.io/app/uploads/2026/04/KubeVirt-Networking-How-to-Preserve-VM-IP-Addresses-During-Migration-2.png" width="1164" /><figcaption class="wp-caption-text" id="caption-attachment-67954">Lift-and-Shift VMs to Kubernetes with Calico L2 Bridge Networks</figcaption></figure>
<h2>Why default KubeVirt networking breaks VM migrations</h2>
<p>When a VM lands in Kubernetes using default pod networking:</p>
<ul>
<li>It receives a <strong>new IP address</strong> from the cluster’s pod CIDR. A range that exists only inside the cluster</li>
<li>The original <strong>VLAN doesn’t exist</strong> inside the cluster. Kubernetes has no native VLAN concept in default networking</li>
<li>Pod IPs are <strong>only meaningful inside the cluster</strong>. The upstream network has no direct visibility into them</li>
</ul>
<p>From the perspective of every system that previously knew the VM by its address, the VM has disappeared. Something with an unfamiliar IP has appeared inside a cluster that the upstream infrastructure can’t see into.</p>
<p>A VM’s IP address accumulates dependencies over time. By the time you’re migrating it, that IP is embedded in:</p>
<ul>
<li><strong>Firewall rules — </strong>security teams wrote rules allowing or denying traffic to that specific address.</li>
<li><strong>DNS records — </strong>the hostname resolves to that IP.</li>
<li><strong>DHCP configuration — </strong>the IP is reserved for that VM’s MAC address.</li>
<li><strong>Monitoring and alerting —</strong> observability tools are configured to watch that address.</li>
<li><strong>Load balancer backends — </strong>upstream load balancers route traffic to that IP.</li>
<li><strong>Application configuration files — </strong>other services have that IP hardcoded.</li>
<li><strong>Compliance and audit documentation —</strong> security posture records reference that IP in that VLAN.</li>
</ul>
<p>VLANs add another dimension. In enterprise environments, VLANs aren’t just a way to segment traffic, they’re security boundaries, designed and owned by the security team. Firewall rules are built around VLAN membership. Compliance frameworks reference VLAN placement. When the VM moves to Kubernetes with default networking, that VLAN disappears. The security boundary is gone.</p>
<p><strong>None of this travels with the VM automatically</strong>. And every broken dependency requires a different team to fix it.</p>
<p>You can see this directly. Running <code>kubectl exec</code> into the virt-launcher pod of a migrated VM shows the interfaces KubeVirt creates with default pod networking:</p>
<pre class="language-yaml code-toolbar line-numbers"><code class="language-yaml">2: eth0@if9: &lt;BROADCAST,MULTICAST,UP,LOWER_UP&gt; mtu 1450
inet 10.60.141.196/32 scope global eth0
3: k6t-eth0: &lt;BROADCAST,MULTICAST,UP,LOWER_UP&gt; mtu 1450
inet 10.0.2.1/24 scope global k6t-eth0
4: tap0: &lt;BROADCAST,MULTICAST,UP,LOWER_UP&gt; mtu 1450 master k6t-eth0</code></pre>
<p style="text-align: center;"><em>eth0 is a Calico-assigned pod CIDR address — meaningful only inside the cluster. k6t-eth0 is KubeVirt’s internal masquerade bridge. tap0 connects to the VM’s virtual NIC. The VM’s original IP is gone. The upstream network sees 10.60.141.196, not the address any firewall rule, DNS record, or application config was written for.</em></p>
<h2>A lift-and-shift becomes a multi-team project</h2>
<p>Here’s what was planned: the platform team moves the VM. One team. The migration is invisible to the rest of the business.</p>
<p>Here’s what actually happens with default pod networking:</p>
<ul>
<li><strong>The IP changes</strong>. The <strong>network team</strong> needs to rewrite firewall rules and update DNS</li>
<li><strong>The VLAN disappears</strong>. The <strong>security team</strong> needs to review the new network placement and approve it</li>
<li><strong>Application config breaks</strong>. The <strong>application team</strong> needs to update config files and hardcoded references</li>
</ul>
<p>Every one of these requires sign-offs, tickets, and coordination</p>
<p>A migration budgeted as a lift-and-shift gets delivered as a network redesign. <strong>Per VM</strong>. At scale, the coordination cost makes migration impractical.</p>
<p>This is where VM migration to Kubernetes stalls, not because the technology doesn’t work, but because the organisational cost exceeds what anyone planned or funded for.</p>
<h2>How to preserve VM IP addresses and VLANs in Kubernetes</h2>
<p>Think about what the problem really is. The VM had a home on the network. A specific IP, a specific VLAN, a specific place in the security model. When it moved to Kubernetes, that home disappeared. Default pod networking gave it a new address in a new network that nothing outside the cluster knows about.</p>
<p>Calico L2 Bridge Networks solve this by doing the opposite. Calico L2 Bridge Networks extend a VM&#8217;s original Layer 2 network segment &#8211; including its IP address, VLAN, and MAC address &#8211; directly into a Kubernetes cluster via a node-level bridge, so the VM&#8217;s network identity survives the migration unchanged. Instead of putting the VM on Kubernetes’s network, it brings the VM’s original network into Kubernetes. The physical VLAN the VM lived on gets extended directly into the cluster via a bridge on the node. The VM connects to that bridge through a secondary interface, and the <a href="http://tigera.io/blog/lift-and-shift-vms-to-kubernetes-with-calico-l2-bridge-networks/">VM preserves its original IP address</a>, the same VLAN, and the same MAC address it had before the migration.</p>
<p>Nothing on the outside knows anything has changed. The firewall still talks to the same IP. DNS still resolves to the right place. The monitoring dashboard still shows the right host. The application that had the IP hardcoded still connects. The security team’s VLAN boundary still exists — it just now exists inside Kubernetes too.</p>
<figure class="wp-caption aligncenter" id="attachment_67955" style="width: 1184px;"><img alt="" class="size-full wp-image-67955" height="850" src="https://www.tigera.io/app/uploads/2026/04/KubeVirt-Networking-How-to-Preserve-VM-IP-Addresses-During-Migration-3.png" width="1184" /><figcaption class="wp-caption-text" id="caption-attachment-67955">L2 Bridge Mode with Calico by Tigera</figcaption></figure>
<p>You can see the difference at the interface level. With Calico L2 Bridge, that same <code>virt-launcher</code> pod now looks like this:</p>
<pre class="language-yaml code-toolbar line-numbers"><code class="language-yaml">2: eth0@if9: &lt;BROADCAST,MULTICAST,UP,LOWER_UP&gt; mtu 1450
   inet 10.60.141.196/32 scope global eth0
3: k6t-eth0: &lt;BROADCAST,MULTICAST,UP,LOWER_UP&gt; mtu 1450
   inet 10.0.2.1/24 scope global k6t-eth0
4: tap0: &lt;BROADCAST,MULTICAST,UP,LOWER_UP&gt; mtu 1450 master k6t-eth0
5: net1: &lt;BROADCAST,MULTICAST,UP,LOWER_UP&gt; mtu 1500
   link/ether 52:54:00:3a:7f:21 brd ff:ff:ff:ff:ff:ff
   inet 10.10.5.42/24 brd 10.10.5.255 scope global net1</code></pre>
<p><code>net1</code> is the secondary interface connected to the L2 bridge that Calico manages on the node. That&#8217;s the VM&#8217;s original IP<code> 10.10.5.42</code>, on its original subnet, with its original MAC address. The pod-side interfaces are still there, KubeVirt still needs them, but the VM&#8217;s actual network identity is preserved on <code>net1</code>. That&#8217;s the interface the rest of your infrastructure talks to.</p>
<h2>Why a secondary interface and not the primary?</h2>
<p>KubeVirt manages the VM’s primary network interface through the <code>virt-launcher</code> pod. That primary interface has two modes: <strong>masquerade</strong> and <strong>bridge</strong>. <strong>Masquerade</strong> NATs all VM traffic through the pod&#8217;s IP. The VM is hidden behind the pod address. <strong>Bridge</strong> mode connects the VM to the pod network bridge. Closer, but still the pod network, not your VLAN.</p>
<p>Neither mode has a way to extend an external VLAN directly to the VM. They’re designed for pod networking, not for preserving legacy network identity.</p>
<p>The secondary interface is what makes this work. Calico attaches an additional interface to the VM and that interface connects to the bridge Calico created on the node, which connects to the trunk carrying your VLAN from the physical switch. The VM’s traffic on that interface goes directly to the right network segment without any translation or tunnelling.</p>
<h2>How Calico sets it up</h2>
<p>The setup is declarative. You define what you want, Calico handles the plumbing.</p>
<p>You create a <code>network</code> resource in Kubernetes that tells Calico which VLAN to bridge and how to map it. Calico reads that and creates the bridge on the node automatically, attaches the trunk interface, and starts tracking the VM&#8217;s IP. A <code>NetworkAttachmentDefinition</code> tells KubeVirt to attach the secondary interface at boot. The <code>VirtualMachine</code> spec references the secondary network, and when the VM starts, <code>net1</code> appears with the right IP.</p>
<p>Migration tools like Forklift (for OpenShift Virtualisation) handle the mapping of existing VM interfaces to the cluster definitions and register the VM’s IP with Calico before migration. From that point, Calico owns the IP, tracking it, keeping routing state correct, and following the VM if it moves between nodes.</p>
<p>Multiple VLANs can run through the same trunk-backed bridge. You don’t need separate infrastructure per VLAN, the same bridge handles them all.</p>
<h2>What you gain after the migration</h2>
<p>Getting the VM into Kubernetes without breaking anything is the primary goal. But once it’s there, a few things become available that weren’t possible in the hypervisor environment.</p>
<h3>Network visibility</h3>
<p>In a traditional hypervisor setup, getting visibility into what a VM is actually doing on the network usually means deploying a separate agent, a network tap, or a dedicated monitoring tool per host. That visibility comes with the unified platform that Calico provides. Calico gives you traffic flow data, communication patterns, and network behaviour for VM interfaces without anything extra to install or manage.</p>
<h3>Security policy you can actually version control</h3>
<p>The firewall rules that protected this VM before migration were probably sitting in a security team’s ticketing system, applied manually to a physical or virtual firewall. They worked, but they weren’t portable, they weren’t reviewable in a pull request, and they weren’t easy to audit.</p>
<p>With Calico, you can express the same security posture as <a href="https://www.tigera.io/learn/guides/kubernetes-security/kubernetes-network-policy/">Kubernetes-native network policy</a>. Labels, selectors, declarative YAML. You don’t have to do this immediately as part of the migration. The VLAN boundary still exists, the existing firewall rules still apply. But when the security team is ready to modernise the policy model, the tooling is already there.</p>
<h3>Live migration that doesn’t touch the network</h3>
<p>Once a VM is running in Kubernetes, it can move between nodes for patching, rebalancing, hardware failures, and the network configuration moves with it. Calico tracks the IP and updates routing state automatically. From the outside, nothing changes. The VM is just on a different node now.</p>
<h2>Making VM migration to Kubernetes practical</h2>
<p>Migration projects fail when the platform team scopes a job as “move the VM” and it turns into “rebuild the network.” That scope creep isn’t a technical failure, it’s what happens when you use a networking model designed for stateless containers to move workloads that were designed around stable, long-lived network identities.</p>
<p>Calico L2 Bridge Networks solve the right problem: keep the network identity intact during the move, let the migration stay within the platform team’s remit, and leave modernisation for when it’s actually planned and funded.</p>
<p><strong>Move now. Modernise later. On your own timeline.</strong></p>
<p><span class="blockquote">Watch our walkthrough to learn more: <a href="http://youtube.com/watch?v=gxpm47mGKPc">Calico L2 Bridge Networking for Virtual Machines</a></span></p>
<p>The post <a href="https://www.tigera.io/blog/kubevirt-networking-how-to-preserve-vm-ip-addresses-during-migration/">KubeVirt Networking: How to Preserve VM IP Addresses During Migration</a> appeared first on <a href="https://www.tigera.io">Tigera – Creator of Calico</a>.</p>
