# Service Networking

Here we will discuss service networking. Previously, we covered pod networking, including how bridge networks are created within each node, how pods get a namespace created for them, how interfaces are attached to those namespaces, and how pods receive IP addresses within the subnet assigned for that node. We also explored how, through routes or other overlay techniques, pods on different nodes can communicate, forming a large virtual network where all pods can reach each other.

## Service as the Preferred Communication Method
Rarely would you configure pods to communicate directly with each other. If a pod needs to access services hosted on another pod, you would always use a service. Let's quickly recap the different kinds of services.
To make the orange pod accessible to the blue pod, we create an orange service. The orange service gets an IP address and a name assigned to it. The blue pod can now access the orange pod through the orange service's IP or its name. We will discuss name resolution later; for now, let's focus on IP addresses.

## Service Accessibility Across Nodes
The blue and orange pods are on the same node. What about access from pods on other nodes? When a service is created, it is accessible from all pods in the cluster, regardless of which nodes the pods are on. While a pod is hosted on a node, a service is hosted across the cluster. It is not bound to a specific node but is accessible only from within the cluster. This type of service is known as ClusterIP.

If the orange pod hosts a database application intended to be accessed only within the cluster, this service type works well.

## Exposing Services Outside the Cluster
For example, if the purple pod hosts a web application and you want to make it accessible outside the cluster, you create another service of type NodePort. This service also gets an IP address and works like ClusterIP, allowing all other pods to access it using its IP. Additionally, it exposes the application on a port on all nodes in the cluster, enabling external users or applications to access the service.

Here we focuses more on services than pods, specifically on how services get their IP addresses, how they are made available across all nodes, and how they are exposed externally through node ports.

## Kubernetes Components Involved in Service Networking
Let's start with a clean slate: a three-node cluster with no pods or services yet. Every Kubernetes node runs a kubelet process responsible for creating pods. Each kubelet watches cluster changes through the Kube API server and creates pods on nodes accordingly. It then invokes the CNI plugin to configure networking for those pods.

Similarly, each node runs kube-proxy, which watches cluster changes through the Kube API server and acts whenever a new service is created. Unlike pods, services are cluster-wide concepts existing across all nodes. In fact, services do not exist as processes or namespaces or interfaces; they are virtual objects without servers listening on their IPs.

## How Services Get IP Addresses and Forward Traffic
When a service object is created in Kubernetes, it is assigned an IP address from a predefined range. The kube-proxy components on each node receive this IP and create forwarding rules on each node, directing any traffic coming to the service IP to the pod's IP. This forwarding is based on IP and port combinations.

Wherever services are created or deleted, kube-proxy updates these rules accordingly.

## kube-proxy Modes and iptables
kube-proxy supports different proxy modes, such as userspace, where it listens on a port for each service and proxies connections to pods by creating IPVS rules. The default and most familiar option is using iptables. The proxy mode can be set during kube-proxy configuration; if not set, it defaults to iptables.

We will examine how iptables are configured by kube-proxy and how to view them on nodes.

## Example: Service IP Assignment and iptables Rules
Consider a pod named db deployed on node one with IP address 10.244.1.2. We create a ClusterIP service to make this pod available within the cluster. Kubernetes assigns the service an IP address, for example, 10.103.132.104. This range is specified in the Kube API server's service cluster IP range option, which by default is 10.0.0.0/24. In my setup, it is set to 10.96.0.0/12, allowing service IPs from 10.96.0.0 to 10.111.255.255.

Similarly, the pod network CIDR range is 10.244.0.0/16, providing pod IPs from 10.244.0.0 to 10.244.255.255. These ranges must not overlap to avoid IP conflicts.

You can see the rules created by kube-proxy in the iptables NAT table output. Search for the service name, as all rules created by kube-proxy include a comment with the service name. These rules mean that any traffic going to the IP address 10.103.132.104 on port 3306 (the service IP and port) should have its destination address changed to 10.244.1.2 on port 3306 (the pod IP and port). This is done by adding a DNAT rule to iptables.

Similarly, when you create a service of type NodePort, kube-proxy creates iptables rules to forward all traffic coming on a port on all nodes to the respective backend pods. You can also observe kube-proxy creating these entries in its logs, which indicate the proxy mode used and when new services are added.

Note that the location of kube-proxy logs may vary depending on your installation, and you may need to check the verbosity level of the process to see these entries.