# Persistent Volume Claims (PVC)

In this lecture, we will discuss Persistent Volume Claims in Kubernetes. Previously, we created a Persistent Volume. Now, we will create a Persistent Volume Claim to make the storage available to a node. Persistent Volumes and Persistent Volume Claims are two separate objects within the Kubernetes namespace.


An administrator creates a set of Persistent Volumes, while a user creates Persistent Volume Claims to use the storage. Once Persistent Volume Claims are created, Kubernetes binds Persistent Volumes to claims based on the request and properties set on the volume. Every Persistent Volume Claim is bound to a single Persistent Volume.


During the binding process, Kubernetes attempts to find a Persistent Volume that has sufficient capacity as requested by the claim, along with matching request properties such as access modes, volume modes, and storage class. If there are multiple possible matches for a single claim and you want to use a particular volume, you can use labels and selectors to bind to the correct volumes.

Note that a smaller claim may be bound to a larger volume if all other criteria match and there are no better options. There is a one-to-one relationship between claims and volumes, so no other claims can utilize the remaining capacity in the volume.


If no volumes are available, the Persistent Volume Claim will remain in a pending state until newer volumes become available to the cluster. Once new volumes are available, the claim will automatically be bound to the newly available volume.

## Creating a Persistent Volume Claim
Let's create a Persistent Volume Claim starting with a blank template. Set the API version to v1 and the kind to PersistentVolumeClaim. Name it "my-claim". Under the specification, set the access modes to ReadWriteOnce and request storage of 500 megabytes.

Use the kubectl create command to create the claim. To view the created claim, run the kubectl get persistentvolumeclaim command. Initially, the claim will be in a pending state.

When the claim is created, Kubernetes checks the previously created volumes. The access modes match, and the capacity requested is 500 megabytes, but the volume is configured with 1 GB of storage. Since no other volumes are available, the Persistent Volume Claim is bound to the Persistent Volume.

Running the get volumes command again shows that the claim is bound to the Persistent Volume we created. To delete a Persistent Volume Claim, run the kubectl delete persistentvolumeclaim command.

## Behavior of Persistent Volumes on Claim Deletion
What happens to the underlying Persistent Volume when the claim is deleted? You can choose the behavior. By default, it is set to retain, meaning the Persistent Volume will remain until manually deleted by the administrator and is not available for reuse by other claims.

Alternatively, the volume can be deleted automatically when the claim is deleted, freeing up storage on the backend storage device. A third option is to recycle the volume, where the data in the volume is scrubbed before making it available to other claims.


# Using PVCs in Pods
Once you create a PVC use it in a POD definition file by specifying the PVC Claim name under persistentVolumeClaim section in the volumes section like this:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: mypod
spec:
  containers:
    - name: myfrontend
      image: nginx
      volumeMounts:
      - mountPath: "/var/www/html"
        name: mypd
  volumes:
    - name: mypd
      persistentVolumeClaim:
        claimName: myclaim
```

The same is true for ReplicaSets or Deployments. Add this to the pod template section of a Deployment on ReplicaSet.


