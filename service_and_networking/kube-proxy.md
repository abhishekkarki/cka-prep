# Kube Proxy

## Why Do We Need kube-proxy?

In Kubernetes, Pods are ephemeral:
- They come and go.
- Their IP addresses change dynamically.

If you want to reach a service (say, a database running in pods), you can’t rely on pod IPs.
Instead, Kubernetes introduces the Service object:
- A stable ClusterIP (virtual IP).
- A way to load balance across multiple pods.


 But: How does traffic sent to that virtual ClusterIP actually get routed to the right pods?
That’s where kube-proxy comes in.

## What is kube-proxy?
- Daemon that runs on every node in the cluster.
- Watches the Kubernetes API server for Service and Endpoints objects.
- Programs the node’s networking rules (iptables, ipvs, or userspace) so traffic to a Service’s ClusterIP/NodePort is routed to a healthy backend pod.
-  kube-proxy = Service load balancer + traffic router inside the cluster.


## Modes of kube-proxy

Depending on setup, kube-proxy can work in different modes:

- **Userspace (legacy, rarely used)**
    - kube-proxy listens on the Service IP.
    - Forwards traffic to one backend pod (round robin).
    - Slow, extra hops → mostly obsolete.

- **iptables (most common today)**
	- kube-proxy installs iptables rules.
	- Traffic to Service IP is NATed directly to one of the pod IPs.
	- Efficient, kernel-level routing.
	- Load balancing is random / probabilistic.

- **IPVS (more advanced)**
	- Uses Linux IPVS (IP Virtual Server).
	- Better performance for high traffic clusters.
	- Supports multiple load balancing algorithms (round-robin, least connections, etc.).


## kube-proxy Responsibilities
*ClusterIP Services*
- Provides the virtual IP for services inside the cluster.
- Example: myservice.default.svc.cluster.local → 10.96.0.10 (ClusterIP).
- kube-proxy ensures packets sent to 10.96.0.10:80 go to one of the pod IPs.
	
*NodePort Services*
- Opens a port on every node (e.g. :30080).
- kube-proxy routes traffic from nodeIP:30080 to service backend pods.
	
*ExternalName Services*
- Not handled by kube-proxy (DNS-based only).

*LoadBalancer Services*
- kube-proxy handles the NodePort side, while the external LB forwards traffic into the cluster.



## Example Flow

Let’s say you have:
```yaml
apiVersion: v1
kind: Service
metadata:
  name: myapp
spec:
  selector:
    app: myapp
  ports:
    - port: 80
      targetPort: 8080
  type: ClusterIP
```

- Service gets a ClusterIP: 10.96.0.15.
- kube-proxy adds iptables rules:

`iptables -t nat -A KUBE-SERVICES -d 10.96.0.15/32 -p tcp --dport 80 -j KUBE-SVC-XYZ`


- KUBE-SVC-XYZ chain randomly selects one backend pod (say 10.244.2.5:8080).
- Traffic is DNAT’d (destination NAT) and routed to that pod.

So when another pod runs:

curl http://10.96.0.15:80

→ It actually hits one of the backend pods without knowing its IP.

⸻

## kube-proxy vs CNI (Important Distinction)
-	CNI plugin (Calico, Flannel, etc.) = pod-to-pod networking, IP address management, routing.
-	kube-proxy = service abstraction, virtual IPs, load balancing.

    They work together:
       - CNI makes sure pods can talk across nodes.
      - kube-proxy makes sure “Services” work as entry points to groups of pods.

⸻

## Limitations
- kube-proxy doesn’t do advanced load balancing (e.g., retries, circuit breaking).
- Doesn’t do L7 routing (that’s where service meshes like Istio/Linkerd step in).
- If kube-proxy is down on a node → services won’t work properly on that node.

⸻

## In summary:
kube-proxy is the service traffic router in Kubernetes.
It makes sure that traffic to a Service’s ClusterIP or NodePort actually gets to the right backend pods, using iptables or IPVS rules.
