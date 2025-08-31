# Persistent Volumes (PV)

In the previous lecture, we learned about volumes. Now, we will discuss persistent volumes in Kubernetes. When we created volumes earlier, we configured them within the pod definition file. This means every configuration detail required to set up storage for the volume was included inside the pod definition file.

## Challenges with Volume Configuration in Large Environments
In large environments with many users deploying numerous pods, each user would have to configure storage every time for each pod. Regardless of the storage solution used, the users deploying the pods must configure storage in all pod definition files within their environment. Whenever changes are necessary, users must update all their pod definitions accordingly.

## Centralized Storage Management with Persistent Volumes
Instead of configuring storage repeatedly in each pod, it is preferable to manage storage more centrally. An administrator can create a large pool of storage, allowing users to carve out pieces from it as required. This is where persistent volumes become useful.

## What is a Persistent Volume?
A persistent volume is a cluster-wide pool of storage volumes configured by an administrator for use by users deploying applications on the cluster. Users can select storage from this pool using persistent volume claims.


## Creating a Persistent Volume
To create a persistent volume, start with the base template and update the API version. Set the kind to `PersistentVolume` and name it `PV-Vol1`. Under the `spec` section, specify the access modes.

## Access Modes
Access modes define how a volume should be mounted on the hosts, such as in read-only mode or read/write mode. The supported values are:
- `ReadOnlyMany`
- `ReadWriteOnce`
- `ReadWriteMany`

## Capacity
Specify the amount of storage to be reserved for this persistent volume. In this example, it is set to 1 GB.

## Volume Type
We will start with the `hostPath` option, which uses storage from the node's local directory. Note that this option is not recommended for production environments.


## Commands to Create and List Persistent Volumes
To create the volume, run the following command:

``` bash
kubectl create -f <persistent-volume-definition.yaml>
```

To list the created persistent volumes, run:
```bash
kubectl get persistentvolume
```

Replace the `hostPath` option with one of the supported storage solutions, such as AWS Elastic Block Store, as discussed in the previous lecture.

## Conclusion
That concludes our lecture on persistent volumes. In the next lecture, we will explore how to use persistent volume claims to claim storage from persistent volumes configured in the cluster.
