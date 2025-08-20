# Container Storage Interface (CSI)

## Introduction to Container Storage Interface (CSI)
Let us now look at the Container Storage Interface. Previously, Kubernetes used Docker alone as the container runtime engine, and all the code to work with Docker was embedded within the 
Kubernetes source code.

With the emergence of other container runtimes such as RKT and CRI-O, it became important to open up and extend support to work with different container runtimes, rather than being dependent solely on the Kubernetes source code. This necessity led to the development of the Container 

## Runtime Interface.
The Container Runtime Interface is a standard that defines how an orchestration solution like Kubernetes communicates with container runtimes such as Docker. In the future, if any new container runtime is developed, it can simply follow the CRI standards and work with Kubernetes without requiring changes from the Kubernetes development team or touching the Kubernetes source code.

Similarly, as seen in networking lectures, to extend support for different networking solutions, the Container Networking Interface was introduced. Any new networking vendor can develop their plugin based on the CNI standards and make their solution compatible with Kubernetes.

The Container Storage Interface was developed to support multiple storage solutions. With CSI, you can write your own drivers for your own storage systems to work with Kubernetes. Examples include Portworx, Amazon EBS, Azure Disk, Dell EMC Isilon, PowerMax, Unity, XtremIO, NetApp, Nutanix, HPE, Hitachi, and Pure Storage, each having their own CSI drivers.

It is important to note that CSI is not a Kubernetes-specific standard. It is designed as a universal standard, and if implemented, it allows any container orchestration tool to work with any storage vendor that supports a CSI plugin. Currently, Kubernetes, Cloud Foundry, and Mesos have adopted CSI.

## How CSI Works
CSI defines a set of remote procedure calls (RPCs) that are invoked by the container orchestrator and must be implemented by the storage drivers. For example, when a pod is created and requires a volume, Kubernetes should call the create volume RPC and pass details such as the volume name.
The storage driver implements this RPC, handles the request by provisioning a new volume on the storage array, and returns the results of the operation. Similarly, when a volume is to be deleted, the container orchestrator calls the delete volume RPC, and the storage driver implements the code to decommission the volume from the array.

The CSI specification details exactly what parameters should be sent by the caller, what should be received by the solution, and what error codes should be exchanged. For those interested, the full CSI specification is available on GitHub at the official URL.