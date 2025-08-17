# Network Policies 

By default in Kubernetes:

- All pods can talk to all other pods across all namespaces.

- All pods can receive traffic from anywhere (inside or outside the cluster).

This is not always desirable in production — e.g., your database pod should not accept traffic from random frontend pods.

A NetworkPolicy is a Kubernetes resource that controls traffic flow (ingress and egress) to and from pods at the IP/network level.
They are enforced by the Container Network Interface (CNI) plugin (Calico, Cilium, Weave, etc.).
If your CNI doesn’t support them, they won’t work.

⸻

## Key Concepts

### Selectors

NetworkPolicies use labels to select pods.
Example:
```yaml
podSelector:
  matchLabels:
    app: backend
```
This means the policy applies to pods with label `app=backend`.

### Policy Types
- **Ingress:** Controls incoming traffic to the selected pods.
- **Egress:** Controls outgoing traffic from the selected pods.

### Rules

Each rule specifies:
- from (for ingress) → who can connect
- to (for egress) → where traffic can go
- ports → which ports/protocols are allowed



## Behavior of NetworkPolicies
- If no NetworkPolicy selects a pod → pod is fully open (default allow).
- If any NetworkPolicy selects a pod → the pod becomes isolated:
- Only traffic explicitly allowed in that policy (or others selecting the pod) is permitted.
- Everything else is denied (default deny).

So applying your first NetworkPolicy often surprises people: traffic suddenly gets blocked.



## Example Scenarios

Example 1: Allow only frontend → backend traffic
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-frontend-to-backend
  namespace: default
spec:
  podSelector:
    matchLabels:
      app: backend
  policyTypes:
    - Ingress
  ingress:
    - from:
        - podSelector:
            matchLabels:
              app: frontend
      ports:
        - protocol: TCP
          port: 80
```
Explanation:
- Selects all pods with app=backend
- Allows ingress only from pods with app=frontend on TCP/80
- Blocks everyone else

⸻

Example 2: Deny all ingress to a pod (Default deny)

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: deny-all-ingress
spec:
  podSelector:
    matchLabels:
      app: db
  policyTypes:
    - Ingress
```
Explanation:
- Selects app=db pods
- No ingress rules defined → denies all inbound traffic

---

Example 3: Allow egress only to DNS

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-dns-egress
spec:
  podSelector:
    matchLabels:
      app: backend
  policyTypes:
    - Egress
  egress:
    - to:
        - namespaceSelector: {}   # any namespace
          podSelector:
            matchLabels:
              k8s-app: kube-dns
      ports:
        - protocol: UDP
          port: 53
```
Explanation:
- Allows backend pods to do DNS lookups (UDP 53)
- Everything else outbound is denied

---

Example 4: Allow traffic between namespaces

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: cross-namespace
  namespace: backend-ns
spec:
  podSelector: {}   # select all pods in backend-ns
  policyTypes:
    - Ingress
  ingress:
    - from:
        - namespaceSelector:
            matchLabels:
              name: frontend-ns
```

Explanation:
- Allows all pods in namespace frontend-ns to connect to all pods in backend-ns


## Common Pitfalls
- CNI must support NetworkPolicies (Calico, Cilium, etc.). If you use Flannel → they won’t work.
- Policies are namespace-scoped. They don’t apply cluster-wide unless you write them in every namespace.
- Default deny happens only if a pod is selected by a NetworkPolicy.
- You may need multiple policies per pod (they are additive).
- If you forget to allow DNS, your apps may suddenly lose connectivity.
- When to Use NetworkPolicies
	- Restrict database access (only backend pods can connect).
	- Prevent compromised pods from scanning the whole cluster.
	- Enforce zero-trust networking.
	- Limit external traffic (egress control).
	- Isolate workloads by namespace or environment (e.g., dev vs prod).


## In summary
NetworkPolicies = firewalls for your pods, defined with labels.
They give you fine-grained control over which pods/namespaces can talk to each other and to the outside world.
