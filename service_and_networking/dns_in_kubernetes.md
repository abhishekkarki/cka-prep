# DNS in Kubernetes

Let's see what names are assigned to which objects, what service DNS records and pod DNS records are, and the different ways you can reach one pod from another.

## Kubernetes Cluster Setup and DNS Scope
Consider a three-node K8s cluster with some pods and services deployed. Each node has a node name and IP address assigned. These node names and IP addresses are likely registered in an organisation's DNS server. 

Let's focus on DNS resolution within the cluster, specifically between components such as pods and services.

Kubernetes deploys a built-in DNS server by default when setting up a cluster. If kubernetes is set up manually, the DNS server must be configured manually as well. Let's see how DNS helps pods reslve other pods and services within the cluster. Nodes are not the focus her.

As long as a cluster networking is correctly set up following best practices, all pods and services can obtain their own OP addresses and communicate with each other. Let's start with a simple example involving two pods and a service.


## Example: Pods and Service Communication
We have a test pod on the left with IP address `10.244.1.5`and a web pod on the right with IP address `10.244.2.5`. Judging by their IPs, they are probably hosted on two different nodes, but this does not affect DNS resolution.
All pods and services can reach each other using their IP addresses within the cluster.
To make the web server accessible to the test pod, we create a service named `web-service`. The service receives an IP address, for example, `10.107.37.188`. When a service is created, the Kubernetes DNS service creates a DNS record mapping the service name to its IP address. Thus, any pod within the cluster can reach this service using its service name.

| Hostname    | Namespace | Type | Root          | IP Address    |
|-------------|-----------|------|---------------|---------------|
| web-service | apps      | svc  | cluster.local | 10.107.37.188 |
| 10-244-2-5  | apps      | pod  | cluster.local | 10.244.2.5    |
| 10-244-1-5  | default   | pod  | cluster.local | 10.244.1.5    | 

So the FQDN is `curl https://10-244-2-5.apps.pod.cluster.local`