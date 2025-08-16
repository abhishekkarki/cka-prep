# CoreDNS in Kubernetes
However, when you have thousands of pods in the cluster and hundreds being created and deleted every minute, this approach is not suitable. Therefore, these entries are moved into a central DNS server. Pods are then pointed to this DNS server by adding an entry into their /etc/resolv.conf file specifying the nameserver IP address of the DNS server, which in this case is 10.96.0.10.

Every time a new pod is created, a record is added in the DNS server for that pod so other pods can access it. The `/etc/resolv`.conf file in the pod is configured to point to the DNS server so the new pod can resolve other pods in the cluster.


## CoreDNS Setup in Kubernetes
CoreDNS is deployed as a pod in the kube-system namespace within the Kubernetes cluster. It runs as two pods for redundancy as part of a replica set within a deployment. For simplicity, we refer to CoreDNS as a pod here.

This pod runs the CoreDNS executable, the same executable used when deploying CoreDNS independently.

CoreDNS requires a configuration file named **`Corefile`**. Kubernetes uses a **`Corefile`** located at `/etc/coredns`. This file contains several plugins configured for handling errors, reporting health, monitoring metrics, caching, and more.

The plugin that enables CoreDNS to work with Kubernetes is the Kubernetes plugin, where the cluster's top-level domain name is set. In this case, it is `cluster.local`. Every DNS record in CoreDNS falls under this domain.

The Corefile is passed into the CoreDNS pod as a ConfigMap object. This allows modification of the configuration by editing the ConfigMap.

With the CoreDNS pod running and configured with the Kubernetes plugin, it watches the cluster for new pods or services. Whenever a pod or service is created, CoreDNS adds a corresponding record in its database.

## Pods pointing CoreDNS
Pods need to point to the CoreDNS server to resolve names. When CoreDNS is deployed, it also creates a service named `kube-dns` by default, making CoreDNS available to other components in the cluster.

The IP address of this service is configured as the nameserver in the pods' DNS configuration. This configuration is done automatically by Kubernetes when pods are created.

The Kubernetes component responsible for configuring the DNS settings on pods is the **`kubelet`**. The kubelet's configuration file contains the IP address of the DNS server and the domain name.

Once pods are configured with the correct nameserver, they can resolve other pods and services. For example, you can access a web service using just `web-service`, `web-service.default`, `web-service.default.svc`, or `web-service.default.svc.cluster.local`.