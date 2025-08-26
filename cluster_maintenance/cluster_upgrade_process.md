# Cluster Upgrade Process

* For checking the current version of the cluster: `kubectl get nodes -o wide` or `kubectl version`
* First check that how many nodes are open to take the workload by verifying the `taints` on the nodes.
* Then check the applications are hosted in the cluster i.e. the deployments
* Check where are the `pods` running in which nodes.
* In order to ensure minimum downtime you should upgrade one node at a time while migrating the workload on the other node.
* To see the latest version available for upgrade with the current version of `kubeadm` tool: `kubeadm upgrade plan`
* 
