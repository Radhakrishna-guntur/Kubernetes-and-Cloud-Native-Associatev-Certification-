# Scheduling

## Labels and Selectors
Labels and selectors provide a standard method to group and filter items based on various criteria. Think of it like sorting species by attributes such as class (mammal, bird, etc.), domestication status, or color. Whether you want to list all green animals or specifically all green birds, labels let you attach key-value pairs to each item, while selectors help retrieve items that meet your criteria.

This concept is applied widely—from tagging YouTube videos or blog posts, to using filters that sort products in an online store.

In Kubernetes, labels and selectors are essential for managing a cluster filled with objects such as Pods, Services, ReplicaSets, and Deployments. Labels help organize these resources by applying characteristics like application type or functionality, allowing you to filter and group them even when dealing with hundreds or thousands of objects. Selectors then query these objects based on the criteria specified by the labels.


<img width="1222" height="685" alt="Screenshot 2025-12-02 at 10 06 46 AM" src="https://github.com/user-attachments/assets/4584e0ff-7835-4f4f-9fc3-88ef1d29d924" />

For every Kubernetes object, you attach labels as needed (for example, app or function) and then use selectors to filter the objects. For instance, if you want to filter objects where the label app equals App1, your selector would look like this:

     app = App1
     
## Specifying Labels in a Pod Definition
In a Pod definition file, labels are defined under the metadata section. Below is an example of how to add labels in a key-value format:

<img width="728" height="675" alt="Screenshot 2025-12-02 at 10 08 07 AM" src="https://github.com/user-attachments/assets/b507e2de-83d4-4b6c-9719-32456e390668" />


## Using Labels and Selectors with ReplicaSets:

When working with ReplicaSets to manage multiple Pods, you label the Pod definition and then specify a selector in the ReplicaSet to group the desired Pods. It’s crucial that the selector in the ReplicaSet specification matches the labels on the Pods exactly.

Note: Only the labels on the Pod template are used for selection; labels on the ReplicaSet itself are not considered.

<img width="727" height="635" alt="Screenshot 2025-12-02 at 10 10 10 AM" src="https://github.com/user-attachments/assets/6f1a14b5-e2a7-435b-b2f0-7527655c0447" />

A single matching label is sufficient. However, if there is a chance that other Pods might share a common label, you should include additional labels in the selector to guarantee that only the intended Pods are selected.

This principle also applies to other Kubernetes objects like Services. When a Service is created, its selector—defined in the Service specification—matches the labels on the Pods (or ReplicaSets) that it targets.

## Annotations in Kubernetes:

While labels and selectors are used for grouping and filtering, annotations offer a way to attach non-identifying metadata to objects for informational purposes. Annotations might include details about the build version, contact information, or other integration data.

For example, consider the following ReplicaSet definition that includes an annotation:


<img width="723" height="678" alt="Screenshot 2025-12-02 at 10 11 15 AM" src="https://github.com/user-attachments/assets/eb309d76-78b2-4e01-9047-32374521c9f9" />

**Summary**
Through the careful use of labels, selectors, and annotations, Kubernetes offers a powerful framework to manage and organize vast numbers of objects effectively. This flexible mechanism improves resource management and simplifies querying, ensuring smooth operations in complex environments.

|
# Taints and Tolerations:

Here we explore how Kubernetes uses taints and tolerations to control the scheduling of pods onto nodes. We will explain the concepts using a clear analogy and provide step-by-step examples to help you understand how to restrict pod placements effectively.

**Understanding the Analogy**

Imagine a bug approaching a person. To prevent the bug from landing, you spray the person with a repellent. In this analogy:

   The repellent represents a **taint** applied to a node.
   The bug corresponds to a **pod** deciding whether to land.
   A bug’s sensitivity or resistance to the repellent represents a **toleration**.
Pods without the appropriate toleration will be repelled by taints, while those with a matching toleration can be scheduled on tainted nodes.

**Translating the Analogy to Kubernetes**

In Kubernetes:

   **Nodes** act as the "person" that is tainted.
   **Pods** act as the "bugs" that may be scheduled based on their tolerations.
   **Taints** on nodes repel pods that do not have matching tolerations.
   **Tolerations** in pods allow them to overcome the taint and be scheduled on that node.

Note that taints and tolerations are solely used for scheduling restrictions, not cluster security.

## Applying Taints on Nodes

You can taint a node using the following command:

     kubectl taint nodes node-name key=value:taint-effect
     
For example, to taint node1 with app=blue and prevent pods from being scheduled on it by default (using the NoSchedule effect), 
run:

    kubectl taint nodes node1 app=blue:NoSchedule

There are three taint effects available:

**NoSchedule:** Pods without the toleration are not scheduled on the node.
**PreferNoSchedule**: The scheduler avoids placing pods on the node, but placement is not strictly prevented.
**NoExecute:** New pods are not scheduled, and existing pods without tolerations are evicted.



<img width="707" height="594" alt="Screenshot 2025-12-02 at 10 18 46 AM" src="https://github.com/user-attachments/assets/234f2034-ae42-4ad8-99d9-230df18a0f29" />

Special Case: NoExecute Effect
Let’s consider the NoExecute taint effect in a dedicated node scenario:

**Imagine three nodes with evenly distributed workloads.**

Node one is tainted with NoExecute, and only pod D has a toleration for this taint.
As a result, any pod on node one without the required toleration, such as pod C, will be evicted, leaving only pod D running on node one.

Moreover, while master nodes are fully capable of running pods, they are usually tainted by default to prevent non-management workloads from being scheduled on them. To view the master node taint, run:

     kubectl describe node <master-node-name>
    
This command displays the taint information that ensures the stability and performance of cluster management services.

## Conclusion
In this guide, we have:

   Explained how taints and tolerations work in Kubernetes using a relatable analogy.
   Demonstrated how to taint nodes and configure tolerations in pod specifications.
   Highlighted the differences between the NoSchedule, PreferNoSchedule, and NoExecute taint effects.
   Discussed best practices for controlling pod placement while noting the additional considerations for using node affinity.

By understanding taints and tolerations, you can effectively manage pod scheduling and optimize your cluster's performance and reliability.

# Node Selectors

Welcome to this guide on Kubernetes **node selectors**. In this article, you'll learn how to restrict pod scheduling to specific nodes based on labels. Imagine a three-node cluster where two nodes have limited hardware resources, and one node is larger and more powerful. To ensure resource-intensive tasks run on the larger node, you can utilize node selectors. By default, Kubernetes schedules pods on any available node, which might inadvertently assign a high-demand pod to a smaller node. Using node selectors, you can guarantee that pods land on nodes that meet the required label criteria.

**Using Node Selectors in Pod Definitions**

To restrict a pod to run only on a node labeled as large, add a **nodeSelector** field to the pod specification. Below is an example demonstrating how to define a pod with a node selector:

      apiVersion: v1
      kind: Pod
      metadata:
         name: myapp-pod
      spec:
        containers:
          - name: data-processor
           image: data-processor
        nodeSelector:
          size: Large

In this example, the **nodeSelector** ensures that the pod is scheduled only on nodes having the label key **size** with the value **Large**.

## Understanding Labels and Node Selectors
The key-value pair (size: Large) in the node selector must match the labels assigned to the nodes. Labels play a critical role in Kubernetes, not only with node selection but also for services, ReplicaSets, and deployments. It is essential to label your nodes accurately before deploying pods that use node selectors.

**Labeling Your Nodes**

Before creating a pod with a node selector, label the appropriate node. Use the following command to add a label to a node:

     kubectl label nodes <node-name> <label-key>=<label-value>

For instance, to label a node named node-1 as large, run:

      kubectl label nodes node-1 size=Large
      
This command sets the label size=Large on node-1, ensuring that any pod requiring this label via its nodeSelector will be scheduled on the correct node.

**Deploying the Pod**
Once your node is labeled and your pod definition includes the node selector, deploy your pod with:

    kubectl create -f pod-definition.yml

This command creates the pod named myapp-pod, which will be scheduled on node-1 thanks to the matching label.

**Note:** Ensure that your node labels are kept up-to-date as your cluster evolves. This helps maintain consistent and predictable pod scheduling.

**Limitations of Node Selectors**

While node selectors are a simple and effective method for basic pod scheduling, they come with limitations. If your scheduling requirements extend beyond a single label match—for instance, if you need to schedule pods on either a large or medium node or exclude nodes labeled as small—node selectors alone won't suffice. For more complex scheduling scenarios, consider using node affinity or anti-affinity. However, for straightforward cases, node selectors provide an easy-to-use solution.


# Node Affinity
Welcome to this comprehensive guide on node affinity in Kubernetes. Node affinity allows you to control the placement of your pods by specifying rules about which nodes are eligible for scheduling. While traditional node selectors provided basic control, node affinity offers advanced scheduling features with flexible operators and expressions.

<img width="698" height="607" alt="Screenshot 2025-12-02 at 10 32 00 AM" src="https://github.com/user-attachments/assets/17cfc4ff-6790-44d3-ad76-136972dc2404" />


## Understanding the Configuration**

**In this configuration:**

    The affinity block is defined under the pod spec.
    nodeAffinity specifies the criteria used for node scheduling.
    The field requiredDuringSchedulingIgnoredDuringExecution indicates a mandatory requirement for scheduling; if no node meets the   criteria, the pod is not scheduled.
    nodeSelectorTerms holds an array of conditions—in this case, ensuring that the node label size must have a value included in the  specified list.

**Behavior and Lifecycle of Node Affinity**

When a pod is created, the Kubernetes scheduler evaluates its node affinity rules to determine the node on which to schedule the pod. However, several scenarios can occur if node conditions change over time.

**Node Affinity Types**
There are two primary types of node affinity currently supported:

**requiredDuringSchedulingIgnoredDuringExecution:**
The scheduler enforces that the pod be placed on a node that satisfies the affinity rules. If no matching node is available, the pod remains unscheduled. Once the pod is running, changes to node labels do not impact the running pod.

**preferredDuringSchedulingIgnoredDuringExecution:**
The scheduler attempts to honor the specified node affinity rules. If a matching node is not found, the pod can be scheduled on a non-matching node. Similarly, node label changes after scheduling are ignored.

## Scheduling vs. Execution
Node affinity rules are applied during two key phases of a pod's lifecycle:

**1.During Scheduling:**
At the time of pod creation, the scheduler evaluates the node affinity rules to determine an appropriate node. If using the required type and no matching node is found (for example, if a node is missing the expected label "Large"), the pod will not be scheduled.

**2.During Execution:**
Once the pod is running, changes in node labels are typically ignored for the current affinity types ("ignored during execution"). However, with forthcoming execution-enforced rules, pods might be evicted if the node subsequently fails to meet the affinity criteria.

**Consider this scenario:**
A pod is scheduled on a node with the label size=Large. If an administrator later removes this label, the pod continues to run under the current behavior. Future implementations with the "required during execution" option could result in pod eviction.

**Conclusion**

In this guide, we broke down the core components of node affinity and demonstrated how various operators and affinity types influence pod scheduling and execution in Kubernetes. Understanding and leveraging these advanced scheduling capabilities allows you to optimize node usage and ensure that pods are placed on nodes that best meet your application requirements.


# Taints and Tolerations vs Node Affinity
Welcome to this lesson. In this guide, we explore how taints and tolerations work alongside node affinity to control pod placement within a Kubernetes cluster. Imagine a scenario with three nodes and three pods, each uniquely identified by their colors: blue, red, and green. The goal is to schedule the blue pod onto the blue node, the red pod onto the red node, and the green pod onto the green node.

Our Kubernetes cluster is shared among multiple teams. Therefore, it is crucial to ensure that no pod from another team is accidentally scheduled on our dedicated nodes, and our pods are not deployed on nodes assigned to other teams.

**Using Taints and Tolerations**
Taints and tolerations are a powerful mechanism to control pod placement. Follow these steps to use them effectively:

**Apply Taints to Nodes:**
Taint each node with a key-value pair corresponding to its color (e.g., blue, red, or green). This marks the nodes and repels any pod that does not have a matching toleration.

**Set Tolerations on Pods:**
Configure each pod with a toleration that matches its designated node’s taint. When pods are created, Kubernetes verifies node taints and only schedules pods that have appropriate tolerations. For example, the green pod, which carries the matching toleration, will only be scheduled on the green node, and the same applies to the blue and red pods.

**Key Consideration**

While taints and tolerations allow pods with the proper tolerations to be scheduled on tainted nodes, they do not enforce that these pods are preferentially scheduled onto these nodes. This means that a pod (like the red pod) could potentially be scheduled on a node that lacks any specific taint if the scheduling criteria permit.

**Using Node Affinity**
Node affinity offers an additional layer of control for scheduling:

**Labeling Nodes:**
Assign each node a label that corresponds to its color (blue, red, or green).

**Setting Node Selectors on Pods:**
Configure each pod with a node selector that matches the node’s label. This enforces that pods only get scheduled on nodes that have the corresponding label, ensuring that pods land on the intended nodes.

However, node affinity alone does not stop other teams' pods from being scheduled on these nodes.

## Combining Taints, Tolerations, and Node Affinity
To fully dedicate nodes to specific pods and prevent external interference, it is best to combine both strategies:

**Prevent External Pod Scheduling:**
Use taints on the nodes and matching tolerations on your pods to ensure that only the correct pods are scheduled on these nodes.

**Enforce Correct Pod Placement:**
Apply node affinity settings to ensure that pods are scheduled strictly on nodes with the appropriate labels.

<img width="678" height="491" alt="Screenshot 2025-12-02 at 10 30 38 AM" src="https://github.com/user-attachments/assets/3f0fa3cd-c921-4e6f-9943-fb89f514ece8" />
