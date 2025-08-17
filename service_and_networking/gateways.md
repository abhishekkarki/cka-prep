# Gatewaty


## What is a Gateway in Kubernetes?
- Ingress was the original way to do HTTP(S) routing into Kubernetes.
- It works, but has limitations:
- Different Ingress Controllers support different annotations.
- Hard to manage multiple listeners (HTTP, TCP, UDP, TLS) in one place.
- Not very extensible.

To solve this, the community designed the Gateway API (a newer, standardized API).
It introduces a richer model with Gateways, Routes, and GatewayClasses.


## Gateway API Key Concepts

GatewayClass
- Cluster-wide template for gateways.
- Defines which implementation is used (e.g., NGINX, Envoy, Istio).
- Similar to how IngressClass works.

Example:
```yaml
apiVersion: gateway.networking.k8s.io/v1
kind: GatewayClass
metadata:
  name: nginx
spec:
  controllerName: k8s.io/ingress-nginx
```


## Gateway
- An instance of a GatewayClass (i.e., the actual load balancer / proxy running in your cluster).
- Defines listeners (ports, protocols, hostnames).

Example:
```yaml
apiVersion: gateway.networking.k8s.io/v1
kind: Gateway
metadata:
  name: my-gateway
  namespace: default
spec:
  gatewayClassName: nginx
  listeners:
  - name: http
    port: 80
    protocol: HTTP
  - name: https
    port: 443
    protocol: HTTPS
```
This says: “Spin up a gateway with HTTP and HTTPS listeners.”



## Routes

Routes bind to Gateways. There are different route types:
- HTTPRoute (like Ingress rules, but richer).
- TCPRoute (for raw TCP traffic).
- UDPRoute.
- TLSRoute.

Example (HTTPRoute):
```yaml
apiVersion: gateway.networking.k8s.io/v1
kind: HTTPRoute
metadata:
  name: my-app-route
  namespace: default
spec:
  parentRefs:
  - name: my-gateway
  hostnames:
  - "myapp.example.com"
  rules:
  - matches:
    - path:
        type: PathPrefix
        value: /app1
    backendRefs:
    - name: app1-service
      port: 80
  - matches:
    - path:
        type: PathPrefix
        value: /app2
    backendRefs:
    - name: app2-service
      port: 80
```
This binds to my-gateway and routes:
 - myapp.example.com/app1 → app1-service
 - myapp.example.com/app2 → app2-service



## How Gateways Compare to Ingress

Feature	Ingress	Gateway API (Gateway + Routes)

Supported protocols	Mostly HTTP/HTTPS	HTTP, HTTPS, TCP, UDP, TLS

Extensibility	Controller-specific annotations	Standard fields, portable

Ownership model	Cluster admins manage ingress controller	Admins create Gateways, teams bind Routes

Multi-tenancy	Limited	Built-in (different teams can attach Routes to shared Gateways)

Maturity	Stable, widely used	Newer (becoming GA in K8s 1.30+), replacing Ingress long-term




## Example End-to-End Flow
- 	Cluster admin installs a GatewayClass (say, NGINX or Istio).
- 	Cluster admin deploys a Gateway with listeners on :80 and :443.
- 	App team deploys their own HTTPRoutes binding to that Gateway.
- 	The Gateway Controller (e.g., NGINX, Istio) configures its proxy accordingly.

 This separation of roles is very powerful: admins control infra, devs control routing.



## Gateways in Service Mesh (Istio / Linkerd)
- In Istio, the old IngressGateway is now modeled via the Gateway API.
- The Gateway API unifies Ingress (north-south traffic) and Service Mesh routing (east-west traffic).
- This means one model for:
- External traffic into the cluster.
- Internal traffic between services.

⸻

## How to Check Gateways in Your Cluster

If you’re on a recent Kubernetes version (≥1.27 usually has CRDs preinstalled):

kubectl get gatewayclasses

kubectl get gateways

kubectl get httproutes

If not, you need to install the Gateway API CRDs:

`kubectl apply -k "github.com/kubernetes-sigs/gateway-api/config/crd?ref=v1.0.0"`




## Typical Use Cases
-	Web routing: Multiple domains, multiple paths → single LB → multiple services.
-	TCP/UDP routing: Databases, custom protocols.
-	TLS termination & passthrough.
-	Multi-team clusters: One Gateway shared, teams attach their own Routes.
-	Service Mesh integration: unify north-south + east-west traffic.


## Summary:
- Ingress was Kubernetes’ first attempt at HTTP routing → works, but limited.
- Gateway API (Gateways, Routes, GatewayClasses) is the modern, extensible replacement.
- Gateways define listeners, Routes define traffic rules, GatewayClass defines which controller implements it.
- Supports HTTP, HTTPS, TCP, UDP, TLS → much richer than Ingress.
