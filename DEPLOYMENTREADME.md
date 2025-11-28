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























