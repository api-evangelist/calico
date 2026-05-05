---
title: "Calico Load Balancer: Simplifying Network Traffic Management with eBPF"
url: "https://www.tigera.io/blog/calico-load-balancer-simplifying-network-traffic-management-with-ebpf/"
date: "Sat, 21 Mar 2026 20:00:55 +0000"
author: "Aadhil Abdul Majeed"
feed_url: "https://www.tigera.io/feed/"
---
<p><strong>Authors:</strong> Alex O’Regan, Aadhil Abdul Majeed</p>
<p>Ever had a load balancer become the bottleneck in an on-prem Kubernetes cluster? You are not alone. Traditional hardware load balancers add cost, create coordination overhead, and can make scaling painful. A Kubernetes-native approach can overcome many of those challenges by pushing load balancing into the cluster data plane. Calico Load Balancer is an <a href="https://www.tigera.io/learn/guides/ebpf/" rel="noopener" target="_blank"><strong>eBPF</strong></a> powered Kubernetes-native load balancer that uses consistent hashing (Maglev) and Direct Server Return (DSR) to keep sessions stable while allowing you to scale on-demand.</p>
<p>Below is a developer-focused walkthrough: what problem Calico Load Balancer solves, how Maglev consistent hashing works, the life of a packet with DSR, and a clear configuration workflow you can follow to roll it out.</p>
<hr />
<h2>Why a Kubernetes-native load balancer matters</h2>
<p>On-prem clusters often rely on dedicated hardware or proprietary appliances to expose services. That comes with a few persistent problems:</p>
<ul>
<li><strong>Cost and scaling friction</strong> &#8211; You have to scale the network load balancer vertically as the size and throughput requirements of your Kubernetes cluster/s grows.</li>
<li><strong>Operational overhead</strong> &#8211; Virtual IPs (VIPs) are often owned by another team, so simple service changes require coordination.</li>
<li><strong>Stateful failure modes</strong> &#8211; Kube-proxy load balancing is stateful per node, so losing an ingress node can break active sessions.</li>
<li><strong>Configuration drift</strong> &#8211; Kubernetes is declarative, but the upstream load balancer is not, which causes divergence over time.</li>
</ul>
<p>Calico Load Balancer flips that model. Instead of dedicated hardware, it uses the <strong>Calico eBPF</strong> data plane on ordinary Linux nodes in the cluster, advertises service IPs via <a href="https://www.tigera.io/blog/when-to-use-bgp-vxlan-or-ip-in-ip-a-practical-guide-for-kubernetes-networking/" rel="noopener" target="_blank">BGP</a>, and makes the load balancing decision consistent across nodes. The result is a system that is cheaper to scale, easier to operate, and more resilient to node or path changes.</p>
<h2>How Calico Load Balancer works (and why Maglev matters)</h2>
<p>The core idea is consistent hashing. Instead of each node picking a backend at random and storing that decision in per-node state, Calico Load Balancer computes the same backend choice on any node for the same flow. This is implemented with Maglev, a consistent hashing algorithm that:</p>
<ul>
<li>Evenly distributes connections across backends.</li>
<li>Minimizes disruption when load balancer nodes come and go.</li>
<li>Allows any load balancer node to make the same backend selection, even mid-connection.</li>
</ul>
<p>Kube-proxy uses random selection plus per-node state, which is fine for many cases but can fail under node churn or route changes. Maglev avoids that by making the decision deterministic. Nodes may still cache the mapping for performance, but the flow-to-backend decision can be reproduced anywhere, which is what keeps sessions stable when traffic lands on a different node.</p>
<p style="text-align: center;"><em><strong>Watch our webinar: <a href="https://www.youtube.com/watch?v=BnuL0V_C3z8">Everything you need to know about Calico Maglev and Kubernetes</a></strong></em></p>
<div style="background-color: #e5e5e5; padding: 25px; margin: 30px 0;">
<h3 style="margin-top: 0; color: #231f20;">Strategic Assessment: Is This Right for Your Deployment?</h3>
<p>Questions you can ask your team to identify if Calico Load Balancer can help your environment:</p>
<ul style="margin-bottom: 0;">
<li>Which services are most impacted by node churn today?</li>
<li>Where do we see the most operational overhead in Virtual IP (VIP) provisioning?</li>
<li>How do we secure access to service VIPs?</li>
<li>Does the network have Equal Cost Multi-Path (ECMP) access to service VIPs?</li>
<li>How do we handle VIP failover?</li>
<li>Are there services with high-throughput requirements?</li>
</ul>
</div>
<h2>The Life of a Packet</h2>
<p>A key design goal is to keep client sessions stable while enabling horizontal scale. Here is a simplified flow for a typical ECMP + BGP setup:</p>
<figure class="wp-caption alignnone" id="attachment_67535" style="width: 1999px;"><a href="https://www.tigera.io/app/uploads/2026/03/image2-1.png"><img alt="This diagram shows how Direct Server Return (DSR) allows the return path to bypass the load balancer node, reducing latency and hop count." class="size-full wp-image-67535" height="1450" src="https://www.tigera.io/app/uploads/2026/03/image2-1.png" width="1999" /></a><figcaption class="wp-caption-text" id="caption-attachment-67535">This diagram shows how Direct Server Return (DSR) allows the return path to bypass the load balancer node, reducing latency and hop count.</figcaption></figure>
<p>A few important details:</p>
<ul>
<li>The top-of-rack router uses ECMP to pick a load balancer node to receive the packet.</li>
<li>That node runs the Maglev algorithm to choose the backend pod. It DNATs the packet and tunnels it to the node that hosts the pod.</li>
<li>The pod replies, and the node SNATs the packet back to the service VIP before it leaves.</li>
<li>With <strong>DSR (Direct Server Return)</strong>, the return path bypasses the load balancer node and goes straight back to the client. The client always sees responses from the advertised service VIP.</li>
</ul>
<p>That <strong>DSR</strong> path is important. It keeps the data path efficient and reduces load balancer hop count on the return path. It also prevents the client from seeing internal pod IPs.</p>
<h3>DSR compared to a traditional return path</h3>
<p>If you have only worked with classic NAT-based load balancers, DSR can feel unusual. The key difference is that the response does not have to traverse the same load balancer node that handled the inbound packet. That has two practical benefits: less work for the load balancer nodes and lower return-path latency.</p>
<h3>Maglev and caching: deterministic and fast</h3>
<p>There are two pieces working together in Calico Load Balancer:</p>
<ul>
<li><strong>The Maglev lookup table:</strong> Provides the deterministic backend choice. Any node can compute the same result for the same flow, which is why mid-connection packets can land on a different node without breaking the session.</li>
<li><strong>A per-flow cache:</strong> (for example, via conntrack) can retain that decision for efficiency, and to preserve existing connections when the backend lookup table changes. It is not the source of truth for correctness.</li>
</ul>
<p>This is a subtle but important difference from kube-proxy. In kube-proxy, the per-node conntrack decision is the only thing tying a flow to a backend. In Calico Load Balancer which uses <a href="https://www.tigera.io/learn/guides/ebpf/" rel="noopener" target="_blank"><strong>Calico’s eBPF dataplane</strong></a>, the decision can be reproduced on any node, which is what makes failover or ECMP rehash events non-disruptive.</p>
<h3>What happens during failures or path changes</h3>
<p>Consistent hashing is not just about distribution. It is about resilience. In practice, you can test this by intentionally re-routing traffic for an existing TCP connection to a different node. Even if the new node has no prior per-flow state, it can recompute the same backend decision using Maglev, so the connection can continue without disruption.</p>
<figure class="wp-caption alignnone" id="attachment_67534" style="width: 1999px;"><a href="https://www.tigera.io/app/uploads/2026/03/image1-1.png"><img alt="" class="size-full wp-image-67534" height="1364" src="https://www.tigera.io/app/uploads/2026/03/image1-1.png" width="1999" /></a><figcaption class="wp-caption-text" id="caption-attachment-67534">Calico uses Maglev consistent hashing to ensure TCP sessions remain stable even if a load balancer node fails or is drained</figcaption></figure>
<p>This matters when:</p>
<ul>
<li>A load balancer node fails or is drained.</li>
<li>ECMP next hops reshuffle due to network outages.</li>
<li>You scale the load balancer pool up or down.</li>
</ul>
<p>Because the decision is deterministic, the packet can land on any node and still find the correct backend. The whole cluster then seemingly acts as a single, distributed load balancer, with per-node caches for additional performance and resilience.</p>
<h2>Configuration workflow (high level)</h2>
<p>Calico Load Balancer is configured and managed declaratively just like any other Kubernetes resource. A typical configuration flow looks like this:</p>
<ul>
<li>Create a dedicated IP pool for Calico LB IPAM, marked for LoadBalancer use.</li>
<li>Create a Service of type LoadBalancer. Calico IPAM allocates a VIP from that pool.</li>
<li>Advertise the VIP to the upstream network using Calico BGP (optional BFD for faster detection of outages).</li>
<li>Ensure your upstream router uses ECMP to send traffic for the VIP to the Calico load balancer nodes.</li>
</ul>
<pre style="background: #f4f4f4; padding: 15px; border-radius: 5px;"># Calico IP pool for load balancer VIPs
apiVersion: projectcalico.org/v3
kind: IPPool
metadata:
  name: loadbalancer-ip-pool
spec:
  cidr: 192.210.0.0/20
  blockSize: 24
  assignmentMode: Automatic
  allowedUses:
    - LoadBalancer
</pre>
<pre style="background: #f4f4f4; padding: 15px; border-radius: 5px;"># Kubernetes Service using Calico LB
apiVersion: v1
kind: Service
metadata:
  name: my-app
  annotations:
    lb.projectcalico.org/external-traffic-strategy: maglev
spec:
  type: LoadBalancer
  selector:
    app: my-app
  ports:
    - port: 443
      targetPort: 8443
</pre>
<p>From there, the VIP is advertised and traffic can arrive through the ECMP paths to any load balancer node. Calico handles the rest.</p>
<h2>Platform Benefits</h2>
<p>The benefits discussion above can translate into real operational advantages for platform teams:</p>
<ul>
<li><strong>Remove Hardware Dependency:</strong> Scale load balancing capacity by adding standard Kubernetes nodes rather than purchasing expensive appliances or coordinating with vendors and avoid vendor lock-in.</li>
<li><strong>Kubernetes-native approach:</strong> Reduces complexity by keeping all service configuration within your existing GitOps workflows &#8211; no separate load balancer management interfaces or external ticketing systems.</li>
<li><strong>Session persistence:</strong> Addresses one of the most common causes of user-facing outages in traditional setups, where losing an ingress node would drop all active connections.</li>
<li><strong>Self-service capability:</strong> Empowers development teams to provision and modify load balancer configurations without waiting for network team approvals, significantly reducing time-to-market for new services.</li>
<li><strong>Predictable traffic distribution:</strong> Maglev&#8217;s consistent hashing ensures that traffic distribution remains predictable and fair even as backend pods scale up and down, preventing the &#8220;hot spot&#8221; issues that can occur with simpler load balancing algorithms.</li>
</ul>
<h2>Conclusion</h2>
<p>Calico Load Balancer gives you a Kubernetes-native way to scale your load balancer and protect critical services without the operational drag of traditional appliances.</p>
<hr />
<div style="background-color: #dceaf3; padding: 25px; border-radius: 8px; margin: 30px 0;">
<h3 style="color: #09287d; margin-top: 0;">Ready to scale your on-prem networking?</h3>
<p>If you want to try this in your environment, here is a safe, incremental path:</p>
<ol>
<li>
<ol>
<li><strong>Identify</strong> a non-critical service that is a good LoadBalancer candidate.</li>
<li><strong>Create</strong> a Calico IP pool for LoadBalancer VIPs and advertise it via BGP to your upstream network.</li>
<li><strong>Enable</strong> a LoadBalancer Service with Maglev for that service and confirm the VIP is reachable.</li>
<li><strong>Validate</strong> failover: remove a load balancer node or change ECMP next hops and verify sessions continue.</li>
<li><strong>Document</strong> the workflow and replicate to other services.</li>
</ol>
</li>
</ol>
<div style="margin-top: 20px;"><a href="https://docs.tigera.io/calico/latest/about/kubernetes-training/about-ebpf" style="background-color: #09287d; color: #ffffff; padding: 12px 24px; text-decoration: none; border-radius: 4px; font-weight: bold; display: inline-block;">Learn more about Calico eBPF</a></div>
</div>
<p>The post <a href="https://www.tigera.io/blog/calico-load-balancer-simplifying-network-traffic-management-with-ebpf/">Calico Load Balancer: Simplifying Network Traffic Management with eBPF</a> appeared first on <a href="https://www.tigera.io">Tigera – Creator of Calico</a>.</p>
