
# Container Orchestration Service Mesh
## Types of Kubernetes Services 

In this article, we will provide an overview of Kubernetes services and explain why they are critical for maintaining consistent application connectivity within a cluster.

Pods are the smallest deployable units in Kubernetes. When running an application, you encapsulate it within a pod. However, pods are ephemeral—they can be created or terminated at any time to align with the desired cluster state managed by Deployments. For applications to function correctly, pods must reliably find and communicate with each other within the cluster.

Each pod receives its own IP address, which enables direct communication. However, because these IP addresses may change frequently, using them directly for component communication (for example, a front-end service communicating with a back-end service) becomes challenging. This is where Kubernetes services are essential.

Kubernetes services provide a stable abstraction over a dynamic set of pods. By leveraging labels, a service can select the appropriate pods from a larger pool, ensuring consistent connectivity even when pods are terminated and recreated. The service is assigned its own IP address, enabling applications to communicate through the service rather than connecting to individual pod IPs.


## There are three primary types of Kubernetes services:

## ClusterIP

This is the default service type. It creates an internal-only service to facilitate communication between applications within the cluster.
The image illustrates three types of Kubernetes services: NodePort, ClusterIP, and LoadBalancer, each represented with a simple diagram.

## NodePort

This service type exposes the application on a specified port on every node in the cluster. It makes the application accessible from outside the cluster, although the access is limited to the node IP addresses and the allocated port.

## LoadBalancer

Supported by specific cloud providers, this service type is similar to NodePort but integrates with an external load balancer. This external component routes traffic to the application’s exposed ports on the nodes, providing a more robust solution for external access.


