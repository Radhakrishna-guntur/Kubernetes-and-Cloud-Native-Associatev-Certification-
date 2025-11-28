## Kubernetes Resources

# Deployments

Kubernetes Deployments build on foundational concepts: Pods encapsulate individual application instances, while ReplicaSets (or ReplicationControllers) manage multiple Pods.
A Deployment is a higher-level construct that not only creates a ReplicaSet but also orchestrates rolling updates, rollbacks, and pause/resume operations.

When deploying an application like a web server, you typically run multiple instances to handle load and ensure uptime. As new versions of your application become available in the Docker registry, 
seamless upgrades are crucial. Since upgrading all instances simultaneously can disrupt active users, Kubernetes Deployments support rolling updates—updating instances one by one. 
Additionally, if an update introduces an error, you can quickly roll back changes. Deployments also allow you to bundle changes—such as updating the web server version, scaling resources, or adjusting resource allocations—and apply them together rather than individually.

**Here is the Deployment YAML**


<img width="622" height="506" alt="Screenshot 2025-11-28 at 6 24 09 PM" src="https://github.com/user-attachments/assets/49bcf04f-780c-4557-b10a-7ce6bbc50668" />


To create the Deployment, save the above YAML content to a file (for example, **deployment-definition.yml**) and run:

     kubectl create -f deployment-definition.yml

Once applied, you can verify the creation of your Deployment with the following commands:


<img width="538" height="676" alt="Screenshot 2025-11-28 at 6 25 36 PM" src="https://github.com/user-attachments/assets/dff3e0c6-788c-4de5-aed8-a4a95cb8c46b" />


## Deployments Rolling Updates and Rollbacks

In this article, you'll learn how to perform **rolling updates, roll back changes, and effectively manage deployment revisions**.

When you create a deployment, Kubernetes automatically triggers a rollout, establishing your initial deployment revision (revision one). Later, when you upgrade your application (for example, by updating the container image version), a new rollout is triggered and a new deployment revision (revision two) is created.


This revision history lets you track changes and enables you to roll back to a previous version if necessary.

You can monitor the status of a rollout by running:

    kubectl rollout status deployment/myapp-deployment

The output might be similar to:

   Waiting for rollout to finish: 0 of 10 updated replicas are available...
   Waiting for rollout to finish: 1 of 10 updated replicas are available...
   ...
   Waiting for rollout to finish: 9 of 10 updated replicas are available...
   deployment "myapp-deployment" successfully rolled out

To view the deployment's revision history, use:

   kubectl rollout history deployment/myapp-deployment

## Deployment Strategies

There are two primary deployment strategies to consider when updating your application:

**1.Recreate Strategy:**

In this approach, all existing replicas are terminated before new ones are created. Although this creates a completely fresh environment, it results in downtime due to the gap between terminating the old pods and starting the new ones.

**2.Rolling Update Strategy:**

This strategy updates instances incrementally. Kubernetes gradually terminates old pods while simultaneously starting new ones, ensuring continuous application availability. Rolling updates are applied by default unless a different strategy is specified during deployment creation.

## Updating a Deployment

There are several ways to update a deployment, such as changing your container image, modifying labels, or adjusting the replica count. If you maintain a deployment configuration file, you can modify it directly. For example:

<img width="570" height="698" alt="Screenshot 2025-11-28 at 6 35 05 PM" src="https://github.com/user-attachments/assets/50a653c0-8673-41ee-827d-3b4cda959bff" />

Note: The kubectl set image command edits the live deployment configuration. This modification may not be reflected in your deployment definition file, so use it with care if you plan to maintain consistency through file-based updates.

## Examining Deployment Details

To inspect detailed information about a deployment—including its strategy—run:

     kubectl describe deployment myapp-deployment

When using the recreate strategy, you will see that the old replica set is scaled down to zero before the new replica set is scaled up. For example:

**How Upgrades Work Under the Hood**

When a deployment is initially created (for example, deploying five replicas), Kubernetes automatically creates a replica set that spawns the required pods. During an upgrade, a new replica set is generated for the updated containers while the pods from the previous replica set are gradually terminated according to the rolling update strategy.

You can verify the changes by listing the replica sets:

   kubectl get replica sets
   
Before the upgrade, the old replica set displays the active pods, and after updating, you will see the new replica set with updated pod counts.

## Rolling Back an Update
If you encounter issues with a new release, Kubernetes allows you to roll back to a previous deployment revision. To perform a rollback, simply execute:

     kubectl rollout undo deployment/myapp-deployment

The output will confirm the rollback:

     deployment "myapp-deployment" rolled back


Before executing the rollback, the new replica set might show all pods (for instance, five replicas) while the old replica set shows none.
After the rollback, these counts are reversed.


<img width="720" height="579" alt="Screenshot 2025-11-28 at 6 39 19 PM" src="https://github.com/user-attachments/assets/625c30a5-4f18-4c99-8688-da0866d8d3fa" />

## Checking Rollout History
Once the deployment is live, view its revision history with:

    kubectl rollout history deployment.apps/myapp-deployment

Example output:

   REVISION   CHANGE-CAUSE
    1          <none>
Since the change cause was not recorded initially, the change cause column is empty.

## Recording the Change Cause
To capture the reason behind changes, delete the existing deployment and re-create it using the --record option:

**Delete the deployment and wait until all pods terminate:**

   kubectl delete deployment myapp-deployment

**Confirm deletion by checking the pods**:

   kubectl get pods

**Re-create the deployment with the record flag:**

   kubectl create -f deployment.yaml --record

**Monitor the rollout status:**

   kubectl rollout status deployment.apps/myapp-deployment

**After completion, verify the revision history:**

   kubectl rollout history deployment.apps/myapp-deployment

The output should now show an entry with a recorded change cause:

     REVISION     CHANGE-CAUSE
        1         kubectl create --filename=deployment.yaml --record=true

## Updating the Deployment with kubectl edit

   kubectl edit deployment myapp-deployment --record
   kubectl rollout status deployment.apps/myapp-deployment
   kubectl rollout history deployment.apps/myapp-deployment

## Using kubectl set image for a Different Update Method
Another approach to update the container image is by using the kubectl set image command. For instance, to update the container image to **nginx:1.18-perl**, execute:

   kubectl set image deployment myapp-deployment nginx=nginx:1.18-perl --record

Then, verify the rollout status:

   kubectl rollout status deployment/myapp-deployment
   
The status messages will indicate that the old replicas for version 1.18 are being replaced by new replicas running version 1.18-perl. Confirm the updated revision history with:

   kubectl rollout history deployment/myapp-deployment

Expected revision history:

    REVISION     CHANGE-CAUSE
      1           kubectl create --filename=deployment.yaml --record=true
      2           kubectl edit deployment myapp-deployment --record=true
      3           kubectl set image deployment myapp-deployment nginx=nginx:1.18-perl --record=true


## Rolling Back to a Previous Revision
If the new image (version 1.18-perl) causes issues, you can easily roll back to a previous version. To revert from revision 3 to revision 2 (running NGINX 1.18), execute:

    kubectl rollout undo deployment/myapp-deployment

Monitor the rollback process:

   kubectl rollout status deployment/myapp-deployment


Once complete, confirm the current deployment configuration:

     kubectl describe deployment myapp-deployment

Note that while the rollout history might show an updated revision number, the state will match the previous, stable revision.

## Simulating a Failed Rollout
To demonstrate how Kubernetes handles a failed rollout, modify the deployment to use a non-existent image. 
Start by editing the deployment:

    kubectl edit deployment myapp-deployment --record

Change the container image to an invalid name, such as nginx:1.18-does-n, as shown below:

<img width="622" height="531" alt="Screenshot 2025-11-28 at 6 54 47 PM" src="https://github.com/user-attachments/assets/9fa1f33b-5c16-4a18-8696-b5aad81b5828" />


Save and exit the editor. At this point, check the rollout status:

    kubectl rollout status deployment/myapp-deployment

In a separate terminal, you can inspect the deployment and pod statuses:

    kubectl get deployments myapp-deployment
    kubectl get pods

You will observe that while five pods continue running with the previous configuration, the new pods trying to run the invalid image show an **"ErrImagePull"** error.

Even with some pods failing, the application remains accessible via the running pods.

## Rolling Back the Failed Deployment
The rollout history now includes a new revision (for example, revision 5) that reflects the failed update. To restore stability (rolling back to revision 4), run:

   kubectl rollout undo deployment/myapp-deployment

Then, monitor the rollback progress:

   kubectl rollout status deployment/myapp-deployment

Finally, verify that all pods are running the correct image version (NGINX 1.18) by checking the pods:

    kubectl get pods

The pod status should look similar to:

             NAME                                         READY   STATUS    RESTARTS   AGE
    myapp-deployment-789c649f95-8s9gk                      1/1     Running   0          12s
    myapp-deployment-789c649f95-9xs8q                      1/1     Running   0          9m5s
    myapp-deployment-789c649f95-dkfm4                      1/1     Running   0          9m7s
    myapp-deployment-789c649f95-qtngw                      1/1     Running   0          9m8s
    myapp-deployment-789c649f95-rktrd                      1/1     Running   0          9m8s
    myapp-deployment-789c649f95-x9jf5                      1/1     Running   0          9m4s

Review the complete rollout history to verify all revisions and their change causes:

   kubectl rollout history deployment/myapp-deployment


## Conclusion

In this lesson, you have learned to:

   Create a Kubernetes deployment using a YAML file.
   Monitor rollout progress using kubectl rollout status.
   Record change causes with the --record flag.
   Update the deployment interactively using kubectl edit and via the kubectl set image command.
   Simulate a failed rollout scenario with an invalid image.
   Roll back to a previous revision using kubectl rollout undo.
 
 This approach ensures that your updates are applied smoothly and, if issues arise, can be quickly reverted to maintain application availability.






