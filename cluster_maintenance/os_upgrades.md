# OS Upgrades

## Introduction to Node Maintenance in Kubernetes
In this lecture, we will discuss scenarios where you might need to take down nodes as part of your cluster, such as for maintenance purposes like upgrading base software or applying security patches. We will explore the options available to handle such cases effectively.

## Impact of Node Downtime on Pods and Applications
Consider a cluster with several nodes and pods serving applications. When one of these nodes goes down, the pods on that node become inaccessible. The impact on users depends on how the pods were deployed. For example, if there are multiple replicas of a pod, users accessing that application are not affected because other replicas continue to serve requests. However, if a pod has no replicas, users accessing that application will be impacted.

## Kubernetes Behavior When a Node Goes Down
If a node comes back online immediately, the kubelet process restarts and the pods on that node come back online. However, if the node remains down for more than five minutes, Kubernetes considers the pods on that node as dead and terminates them. If these pods are part of a replica set, Kubernetes recreates them on other nodes.
The time Kubernetes waits for a pod to come back online is called the pod-eviction-timeout, which is set on the controller manager with a default value of five minutes. When a node goes offline, the master node waits up to five minutes before marking the node as dead.
When the node returns online after the **pod-eviction-timeout**, it comes up without any pods scheduled on it. For pods that were part of a replica set, new pods are created on other nodes. However, pods not part of a replica set are lost.

## Maintenance Strategies for Nodes
If you need to perform maintenance on a node, and you know that the workloads running on it have replicas, and it is acceptable for them to be down briefly, you can perform a quick upgrade and reboot, assuming the node will come back online within five minutes. However, since it is uncertain whether the node will return within that time or at all, a safer approach exists.

## Draining and Cordoning Nodes
You can purposefully drain a node of all workloads so that the pods are gracefully terminated on that node and recreated on other nodes. Draining also cordons the node, marking it as unschedulable, which means no new pods can be scheduled on it until the restriction is removed.
Once the pods are safely running on other nodes, you can reboot the drained node. When it comes back online, it remains unschedulable. You must uncordon the node to allow pods to be scheduled on it again. Note that pods moved to other nodes do not automatically move back to the original node; if pods are deleted or new pods are created, they may be scheduled on the uncordoned node.
Apart from the `drain` and `uncordon` commands, there is also the `cordon` command. Cordon marks a node as unschedulable but does not terminate or move existing pods. It simply prevents new pods from being scheduled on that node.

## Summary
- Draining a node gracefully moves workloads to other nodes and marks the node unschedulable.
- Cordoning a node only marks it unschedulable without affecting existing pods.
- After maintenance, uncordon the node to allow scheduling of pods again.
- Practice using the `drain`, `cordon`, and `uncordon` commands to manage node maintenance effectively.



## Exercises
- We need to take `node01` out for maintenance. Empty the node of all applications and mark it unschedulable.
  - `kubectl drain node01 --ignore-daemonsets`: evict all apart from the daemonsets

- The maintenance tasks have been completed. Configure the node node01 to be schedulable again.
  - `kubectl uncordon node01`

- If the pod is not a part of the `replicaset`, the `kubectl drain` command will not work you need to use the `--force` flag and now this pod will be deleted forever.
- Therefore in these kind of scenarios we can use `kubectl cordon node01` now no new pods can be scheduled in this node and existing pods will not be affected by this operation.
- 
   



