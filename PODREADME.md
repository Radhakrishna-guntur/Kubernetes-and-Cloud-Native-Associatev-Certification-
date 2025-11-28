# Kubernetes and Cloud Native Associate - KCNA

## Kubernetes Resources

## Pods

Welcome to this comprehensive guide on Kubernetes pods. In this article, we will explore the essentials of pods, their role in container orchestration, 
and how they simplify scaling and deployment. Before proceeding, ensure you have:

An application developed, built into Docker images, and published to a Docker repository (e.g., Docker Hub) for Kubernetes to pull.

A fully operational Kubernetes cluster (either single-node or multi-node) up and running.

All additional services required by the application are active.

Kubernetes uses pods as the smallest deployable units that encapsulate one or more containers. 
A pod represents a single instance of your application running on a worker node.

**The Role of Pods:**

In a basic scenario, you may run a single instance of your application in a Docker container encapsulated within a pod on a single-node cluster. 

However, as the number of users grows, simply adding more containers inside a single pod is not the solution.

When demand increases and scaling is necessary, you should deploy additional pods, each containing an instance of your application.

As load increases, more pods are created on the same node or on additional nodes, ensuring efficient distribution of resources and optimal performance.

Another important aspect is that pods typically maintain a one-to-one relationship with containers. To scale up, create new pods; to scale down, delete existing pods. 

Additional containers are not added to an existing pod for scaling purposes.


<img width="975" height="511" alt="Screenshot 2025-11-28 at 11 10 26 AM" src="https://github.com/user-attachments/assets/86bf5a28-7a80-4ba6-8fee-e45bc485850f" />


## Multi-Container Pods

While most pods run a single container, there are scenarios where running multiple containers together is beneficial. 
For instance, a helper container may run alongside your primary application container to support functions like data processing or file uploads. 
In this setup, both containers start and terminate together, sharing the same network namespace and storage volumes.

The diagram below demonstrates a multi-container pod configuration:


<img width="958" height="458" alt="Screenshot 2025-11-28 at 11 11 16 AM" src="https://github.com/user-attachments/assets/19a2f890-458c-48b9-a9d0-923cb904e49e" />



Manual configurations for linking containers in Docker can lead to significant operational overhead. 
Pods in Kubernetes abstract these complexities, offering streamlined management and deployment.


**Deploying a Pod Using kubectl**

Deploying a pod in Kubernetes is straightforward with the kubectl command-line tool. 
For example, to create a pod running the nginx Docker image, execute:

**kubectl run nginx --image nginx**

This command pulls the nginx image from Docker Hub by default. For private repositories, additional configurations may be applied.

After deployment, verify the pod status with:

**kubectl get pods**

Initially, you may see output similar to:

C:\Kubernetes>**kubectl get pods**

      NAME                           READY             STATUS                   RESTARTS         AGE

      nginx-8586cf59-whssr           0/1             ContainerCreating             0             3s**



After a short period, the status will update to indicate that the pod is running:

**C:\Kubernetes>kubectl get pods**


       NAME                   READY   STATUS    RESTARTS      AGE
       nginx-8586cf59-whssr   1/1     Running     0           8s

At this stage, note that the nginx web server is configured for internal access only.
To expose it to end users, additional networking and service configurations are required.


# Pod Operations


**Creating a Pod**

To create a pod named "nginx" using the Docker image "nginx" (pulled from Docker Hub), run the following command:

     kubectl run nginx --image=nginx


Once this command executes, Kubernetes creates the pod. You can verify its creation and status by checking the list of pods:

    # kubectl run nginx --image=nginx
      pod/nginx created


    # kubectl get pods
      NAME    READY   STATUS    RESTARTS   AGE
      nginx   1/1     Running   0          3s

The output confirms that the pod is running. The "READY" column indicates how many containers are in a ready state, 
"RESTARTS" shows the number of times the container has restarted, and "AGE" reflects how long the pod has been active.

**Inspecting Pod Details**

For more in-depth information about the pod—including its labels, node assignment, internal IP address, and container specifics—use the kubectl describe command:
    
      # kubectl describe pod nginx
       Name:           nginx
       Namespace:      default
       Priority:       0
       Node:           minikube/192.168.99.100
      Start Time:     Sat, 11 Jul 2020 00:49:39 -0400
      Labels:         run=nginx
      Annotations:    <none>
      Status:         Running
      IP:             172.17.0.3
       IPs:
           IP: 172.17.0.3
      Containers:
         nginx:
         Container ID:   docker://987785b312ad2e38c77132300f8709b8a027566462c2d18634ff13b34
         Image:          nginx
         Image ID:       docker-pullable://nginx@sha256:a23a9056789b968a186c5205f4



This command output provides essential metadata and status details such as the pod's start time, the node it is running on, and its internal IP address (172.17.0.3). If multiple containers were running within the pod, each would be listed under the "Containers" section.


**Viewing Pod Information in Wide Format**

For a summary that also includes node and internal IP details, use the following command:

**kubectl get pods -o wide**

The output will resemble:


     NAME    READY   STATUS    RESTARTS   AGE     IP            NODE        NOMINATED NODE
     nginx   1/1     Running   0          2m28s   172.17.0.3    minikube    <none>


Here, each pod is assigned its own internal IP address (in this case, 172.17.0.3), which enables network communications within the cluster.



# Pods with YAML

Hello, and welcome to this tutorial on creating a Pod using a YAML-based configuration file. In this guide, you will learn how to write and apply YAML files specifically tailored for Kubernetes. Kubernetes leverages YAML files to create various objects like Pods, ReplicaSets, Deployments, and Services. Each definition file adheres to a common structure containing four top-level fields: **apiVersion, kind, metadata, and spec.**

## Understanding the Key Fields

Every Kubernetes YAML definition file must include the following properties:

**1. apiVersion**
This property specifies the version of the Kubernetes API you are using to create your object. For Pods, you typically specify "v1". For other objects like Deployments, you might use "apps/v1" or "extensions/v1beta1".

**2. kind**
The kind field defines the type of object being created. In this lesson, we will focus on creating a Pod, so you would set kind as "Pod". Other common values include ReplicaSet, Deployment, or Service.

**3. metadata**
The metadata section contains identifying information about the object, such as its name and labels. It is a dictionary rather than a string. Ensure all keys (like name and labels) have proper and consistent indentation. 

**4. spec**
The spec section provides detailed configuration for the object. For a Pod, this involves specifying one or more containers that will run within it. The key containers holds a list of container definitions; each item in the list is a dictionary containing that container's configuration.

Below is a complete example of defining a Pod using YAML:

<img width="686" height="297" alt="Screenshot 2025-11-28 at 12 52 36 PM" src="https://github.com/user-attachments/assets/7b4222fb-f13a-4271-9f9e-98c225c019dd" />



**In this snippet:**

**name** uniquely identifies the Pod.
**labels** are key-value pairs used to categorize and manage Pods based on roles such as front-end, back-end, or database.

The **containers** key defines a list of container specifications.
The dash before **name** indicates the first (and only) container in the list.
The container is named **nginx-container** and it uses the Docker image **nginx**.

## Deploying Your Pod

After creating your YAML configuration file (e.g., **pod-definition.yml**), you can deploy the Pod using the following command:

    kubectl create -f pod-definition.yml

Kubernetes will then create the Pod as defined in your file.

**Checking Pod Status**

To verify that your Pod is running, use:

       kubectl get pods





**Conclusion**

Pods are fundamental to Kubernetes, encapsulating containers and simplifying scaling, deployment, and management. 
Whether running a single container or a multi-container setup, the pod abstraction provides a robust foundation for building scalable, 
resilient applications on your Kubernetes cluster.

In summary, when creating a YAML file for Kubernetes, always include the four top-level properties:

      apiVersion: Specifies the API version.
      kind: Defines the type of Kubernetes object.
      metadata: Holds essential information such as name and labels.
      spec: Details the configuration specifics, such as container settings in a Pod.
      
