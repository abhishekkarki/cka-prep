# Rolling Updates and Rollback

## Introduction to Updates and Rollbacks
In this lecture, we will discuss updates and rollbacks in a deployment. Before exploring how to upgrade our application, it is important to understand rollouts and versioning in a deployment.
When you first create a deployment, it triggers a rollout. A new rollout creates a new deployment revision, which we will call revision one. In the future, when the application is upgraded—meaning when the container version is updated to a new one—a new rollout is triggered, and a new deployment revision is created, named revision two. This process helps us keep track of the changes made to our deployment and enables us to roll back to a previous version of the deployment if necessary.
You can check the status of your rollout by running the command `kubectl rollout status` followed by the name of the deployment. To see the revisions and history of rollouts, run the `kubectl rollout history` command followed by the deployment name. This will display the revisions and history of our deployment.

## Deployment Strategies
There are two types of deployment strategies. For example, suppose you have five replicas of your web application instance deployed. One way to upgrade these to a newer version is to destroy all of them and then create newer versions of application instances. This means first destroying the five running instances and then deploying five new instances of the new application version.
The problem with this approach is that during the period after the older versions are down and before any newer version is up, the application is down and inaccessible to users. This strategy is known as the recreate strategy. Thankfully, this is not the default deployment strategy.
The second strategy is where we do not destroy all instances at once. Instead, we take down the older version and bring up a newer version one by one. This way, the application never goes down, and the upgrade is seamless. Remember, if you do not specify a strategy while creating the deployment, it will assume the rolling update strategy. In other words, rolling update is the default deployment strategy.

## Updating a Deployment
We have discussed upgrades. Now, how exactly do you update your deployment? Updating could involve different things such as updating your application version by updating the version of Docker containers used, updating their labels, or updating the number of replicas, among others.
Since we already have a deployment definition file, it is easy for us to modify this file. Once we make the necessary changes, we run the `kubectl apply` command to apply the changes. A new rollout is triggered, and a new revision of the deployment is created.
Alternatively, you could use the `kubectl set image` command to update the image of your application. However, doing it this way will result in the deployment definition file having a different configuration, so you must be careful when using the same definition file to make changes in the future.
Differences Between Recreate and Rolling Update Strategies

The difference between the recreate and rolling update strategies can also be seen when you view the deployments in detail. Run the `kubectl describe deployment` command to see detailed information regarding the deployments.
You will notice that when the recreate strategy was used, the events indicate that the old replica set was scaled down to zero first, and then the new replica set scaled up to five. However, when the rolling update strategy was used, the old replica set was scaled down one at a time, simultaneously scaling up the new replica set one at a time.
How Deployment Upgrades Work Under the Hood
When a new deployment is created, for example, to deploy five replicas, it first creates a replica set automatically, which in turn creates the number of pods required to meet the number of replicas.
When you upgrade your application, as we saw earlier, the Kubernetes deployment object creates a new replica set under the hood and starts deploying the containers there, while simultaneously taking down the pods in the old replica set following the rolling update strategy.
This behavior can be observed when you list the replica sets using the `kubectl get replicasets` command. Here, you will see the old replica set with zero pods and the new replica set with five pods.

## Rolling Back a Deployment
Suppose after upgrading your application, you realize something is wrong with the new version of the build you used. You would like to roll back your update. Kubernetes deployments allow you to roll back to a previous revision.
To undo a change, run the `kubectl rollout undo` command followed by the name of the deployment. The deployment will then destroy the pods in the new replica set and bring the older ones up in the old replica set, restoring your application to its previous state.
When you compare the output of the `kubectl get replicasets` command before and after the rollback, you will notice the difference. Before the rollback, the first replica set had zero pods and the new replica set had five pods. After the rollback is finished, this is reversed.
