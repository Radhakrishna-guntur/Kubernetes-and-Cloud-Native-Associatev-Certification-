# Kubernetes Resources

## ReplicaSets
In this lesson, we explore the concept of replicas in Kubernetes and the importance of replication controllers for ensuring high availability. Imagine a scenario where your application runs only one pod. If that pod crashes, users lose access to your application. To avoid downtime, it's critical to run multiple instances (or pods) simultaneously. A replication controller guarantees that the desired number of pods are always running in your cluster, delivering both high availability and load balancing.

Even if you intend to run a single pod, the replication controller automatically initiates a new pod if the existing one fails. Whether you need one pod or a hundred, the replication controller maintains that number, distributing load across multiple instances. For example, if your user base grows, additional pods can be deployed. In cases where one node runs out of resources, new pods can automatically be scheduled on other nodes.

<img width="688" height="382" alt="Screenshot 2025-11-28 at 2 28 49 PM" src="https://github.com/user-attachments/assets/368dd376-4a2f-4c56-9b3f-b910bc271cec" />

## Creating a ReplicationController
To create a ReplicationController, start by defining a configuration file named rc-definition.yaml. Like any Kubernetes definition file, it includes the following sections: API version, kind, metadata, and spec.

API Version: For a ReplicationController, use v1.
Kind: Set it as ReplicationController.
Metadata: Provide a unique name (for example, myapp-rc) along with labels that categorize your application (such as app and type).
Spec: Define the desired state of the object:
Specify the number of replicas.
Include a template section for the pod definition. (Note: do not include the API version and kind from the original pod file; include only the pod’s metadata, labels, and spec, indented as a child of the template.)

Below is an example ReplicationController definition:


<img width="348" height="531" alt="Screenshot 2025-11-28 at 2 30 48 PM" src="https://github.com/user-attachments/assets/e506765b-24df-441b-8970-a2a6b4c9ebc3" />



Once the file is ready, create the replication controller with:

    kubectl create -f rc-definition.yaml
    
After creation, verify the replication controller details using:

     kubectl get replicationcontroller

This command displays the desired number of replicas, the current number, and the count of ready pods. To list the pods created by the replication controller, run:

       kubectl get pods
       
You will notice that the pod names begin with the replication controller’s name (e.g., myapp-rc-xxxx), indicating their automatic creation.

# Introducing ReplicaSets

A ReplicaSet functions similarly to a ReplicationController but comes with key differences:

**1.API Version and Kind:**

   For a ReplicaSet, set the API version to **apps/v1** (instead of v1).
   The kind should be **ReplicaSet**.
   
**2.Selector Requirement:**

A ReplicaSet demands an explicit selector in its configuration. Typically defined under matchLabels, the selector identifies which pods the ReplicaSet will manage. This feature also enables the ReplicaSet to adopt existing pods that match the specified labels, even if they were not created by it.
Below is an example of a ReplicaSet definition file named **replicaset-definition.yml**:


<img width="373" height="608" alt="Screenshot 2025-11-28 at 2 34 56 PM" src="https://github.com/user-attachments/assets/e98009a5-c4a2-4f7b-8cb6-6ed5e52ce9c9" />


**Create the ReplicaSet using:**

     kubectl create -f replicaset-definition.yml

**Then verify the ReplicaSet and its pods:**

     kubectl get replicaset
     kubectl get pods

The ReplicaSet monitors pods with matching labels. If pods already exist with these labels, the ReplicaSet will adopt them rather than immediately creating new ones. Nevertheless, the template remains essential for creating new pods if any managed pod fails.

## Labels, Selectors, and Their Role

Labels are vital for organizing and selecting subsets of objects in Kubernetes. When deploying multiple instances (for instance, three pods for a front-end application), a ReplicaSet uses labels and selectors to manage these pods effectively. Consider the following sample configuration that highlights the relationship between pod labels and the ReplicaSet selector:

<img width="445" height="300" alt="Screenshot 2025-11-28 at 2 37 15 PM" src="https://github.com/user-attachments/assets/e5697af9-ef06-4013-9d79-ae2fe4fe8b5a" />


The ReplicaSet uses its selector to monitor and manage pods with the label **tier: front-end**. 
This concept of labels and selectors is widely used throughout Kubernetes to maintain order and efficiency.


## Scaling ReplicaSets:

Scaling ReplicaSets allows your application to adapt to changing demand. Suppose you started with three replicas and later need to scale up to six. There are multiple approaches:

**1.Edit the Definition File:**
Update the replicas field in your ReplicaSet definition file to six, then run:

    kubectl replace -f replicaset-definition.yml

**2.Use the Scale Command:**

Alternatively, use the kubectl scale command:

     kubectl scale --replicas=6 -f replicaset-definition.yml

Or, if you prefer using the ReplicaSet name:
     kubectl scale --replicas=6 replicaset/myapp-replicaset

<img width="667" height="633" alt="Screenshot 2025-11-28 at 2 40 06 PM" src="https://github.com/user-attachments/assets/f9047a03-4db7-4354-9c13-2df56343a2c5" />


<img width="675" height="542" alt="Screenshot 2025-11-28 at 2 43 00 PM" src="https://github.com/user-attachments/assets/dc7d46c4-17a7-42cc-bc56-2854b181df25" />


Understanding how labels, selectors, and ReplicaSets interact is crucial to maintain high availability and scalable deployments in your Kubernetes environment. This lesson has covered the fundamentals of replication controllers and ReplicaSets, along with their creation, updating, and scaling procedures.

<img width="973" height="513" alt="Screenshot 2025-11-28 at 2 45 02 PM" src="https://github.com/user-attachments/assets/29b94dd3-db2b-46ed-ae62-06f1844bd690" />


This concludes our discussion on ReplicaSets. Apply these concepts to ensure robust and scalable deployments in your Kubernetes cluster.

