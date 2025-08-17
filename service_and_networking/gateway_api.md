# Gateway API

## Introduction to Gateway API
Let's now look at an introduction to Gateway API. Earlier, when we spoke about Ingress, we discussed two services sharing the same Ingress resource. What if each service was managed by different teams or even completely different organizations or businesses? For example, what if the wear service was managed by Team A and the video service was managed by Team B? In such a case, the underlying Ingress resource is still a single resource, which can only be managed by one team at a time. So, in a multi-tenant environment, Ingress can pose a challenge. Teams would need to coordinate handling the same Ingress resource, which might lead to conflicts. Ingress lacks sufficient support for multi-tenancy.

Another limitation of Ingress is the options for rules configuration. Ingress only supports a CTP-based ruleset, such as host matching or path matching. Other features like TCP/UDP routing, traffic splitting, header manipulation, authentication, rate limiting, and others are not currently supported. These configurations are handled by the controllers and are passed through to the controllers using annotations. For example, you can see some NGINX configurations passed through, such as SSL and redirects. As a result, you will see complex annotations specified in different Ingress rules, such as configuring CORS with NGINX-specific settings or Traefik-related configurations.

The challenge here is that these configurations are very specific to the underlying controllers, such as NGINX and Traefik respectively. Kubernetes itself is not aware of these settings, so it cannot validate if they are correct or incorrect. These configurations are merely passed to the underlying controllers. Therefore, different configurations exist for different controllers for the same use case, and these configurations can only be used with these specific controllers.

## Gateway API Overview
This is where Gateway APIs come in. Gateway API is an official Kubernetes project focused on layer four and layer seven routing. This project represents the next generation of Kubernetes Ingress load balancing and service mesh APIs. One of the challenges we discussed with Ingress was the lack of support for multi-tenancy. With different teams accessing the same infrastructure, how do you provide the flexibility needed by the users while maintaining control by the owners of the infrastructure? Gateway API addresses this by introducing three separate objects that are managed by three separate personas.

## The infrastructure providers configure the GatewayClass.
The GatewayClass defines what the underlying network infrastructure would be, such as NGINX, Traefik, or other load balancers.

The cluster operators configure the Gateway, which are instances of the GatewayClass.
The application developers create the HTTPRoutes.

Unlike Ingress where we only had HTTPRoutes, Gateway API supports other route types such as TCPRoute and GRPCRoute. Let's look at how each of these are created.

## GatewayClass Example
The GatewayClass has the API version gateway.networking.k8s.io/v1, the kind is GatewayClass, the name is example-class, and the controller name is gateway-controller. Like Ingress, we must also deploy a controller for Gateway. The controller name specified here must match the controller deployed.

## Gateway Object Example
The Gateway object has the API version gateway.networking.k8s.io/v1, kind Gateway, name example-gateway, and it specifies the GatewayClass created above. It also configures HTTPListeners with the name http on port 80.

## HTTPRoute Example
The HTTPRoute has the API version gateway.networking.k8s.io/v1, kind HTTPRoute, and name example-httproute. It references the parent Gateway example-gateway. It matches requests with the host name www.example.com and a path rule for /login. The backend 
service it refers to is example-svc on port 8080.

In this example, HTTP traffic from the Gateway example-gateway with the host header set to www.example.com and the request path /login will be routed to the service example-svc on port 8080.

The HTTPRoute we saw is a layer-seven protocol. Additional route options available include TLSRoute, TCPRoute, UDPRoute, GRPCRoute, and others.

## Addressing Ingress Limitations with Gateway API
Let's revisit some limitations we saw with Ingress and see how they are configured in Gateway API. Earlier, we saw that the basic TLS configuration in Ingress goes in the spec.tls section. To ensure all traffic uses HTTPS, we need to redirect HTTP to HTTPS using NGINX-specific annotations. These annotations won't work with other Ingress controllers.

The Gateway API approach is much more declarative and structured. Everything is defined in the proper spec, with no annotations needed. The listeners section clearly shows setting up an HTTPS endpoint on port 443. The TLS configuration is explicit with the mode set to terminate, indicating TLS termination at the Gateway. The certificateRefs directly reference the TLS secret. The allowedRoutes specify which kinds of routes can attach to this listener, in this case, HTTPRoutes.
Here's another example. Through NGINX-specific annotations, we might say this is a canary deployment and send 20% of the traffic there, with the remaining 80% going to the primary Ingress. However, this is not obvious from the configuration alone and only works with NGINX. Other controllers might not understand these annotations because they are specific to NGINX.

Gateway API tells the complete story in one clear configuration. Everything is visible in one place with no hidden primary configuration needed. We can see both services at versions v1 and v2 in the backendRefs section. The traffic split is explicitly defined: 80% goes to v1 and 20% goes to v2. No annotations are needed; this is a native feature. This will work the same way across any Gateway API implementation.
These configurations work with any controller and are not specific to Ingress or any particular implementation.

Here's yet another example. Earlier, we saw complex controller-specific annotations needed for advanced configurations like CORS settings. Now, with Gateway API, we can configure these centrally with no annotations needed. Everything is defined in the spec. CORS headers are explicitly defined using the responseHeaderModifier filter. The configuration is more readable and self-documenting, and it will work consistently across any Gateway API implementation.

## Gateway API Adoption
Most controllers have now implemented Gateway API or are on the way to implement it. Amazon EKS, Azure Application Gateway for Containers, Contour, Easegress, Envoy Gateway, Google Kubernetes Engine, HAProxy Kubernetes Ingress Controller, Istio, Kong, Kuma, NGINX Gateway Fabric, and Traefik Proxy are already generally available with the implementation, and others are on their way.
Let's conclude this lecture and head over to the labs to practice working with Gateway API controllers.