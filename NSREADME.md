
## Kubernetes Resources

# Kubernetes Namespaces

Namespaces in Kubernetes are a powerful tool to isolate resources, manage policies, and efficiently organize your cluster environment—whether for development, testing, or production. 
By leveraging namespaces, you can deploy applications securely and prevent accidental cross-environment interference.

 Understanding namespaces is essential for managing resources and policies within a single Kubernetes cluster, especially in production environments. In this article, we use an everyday analogy to clarify the role and importance of namespaces.

**Default and System Namespaces**

So far, you have been creating objects like Pods, Deployments, and Services in a single namespace—the default namespace, which is automatically created when the cluster is set up. In addition, Kubernetes creates several other namespaces at startup:

**kube-system:** Contains critical Pods and Services (such as networking solutions and DNS) that are isolated from the user to prevent accidental modifications.
**kube-public:** Contains resources that should be accessible to all users.
For smaller or learning environments, you might continue working in the default namespace. However, in enterprise or production setups, namespaces become crucial. For instance, if you have both development and production environments sharing the same cluster, isolating resources via separate namespaces can prevent accidental interference.


**Remember, if you want to list Pods across all namespaces, use:**

   kubectl get pods --all-namespaces

## Creating Pods in Specific Namespaces

      kubectl create -f pod-definition.yml --namespace=dev

<img width="721" height="557" alt="Screenshot 2025-11-28 at 7 07 03 PM" src="https://github.com/user-attachments/assets/e234c62c-a962-4c64-8dd8-725ad0fcf75a" />

<img width="728" height="661" alt="Screenshot 2025-11-28 at 7 07 43 PM" src="https://github.com/user-attachments/assets/efbfe178-79a7-49f5-864e-22fcef44ad5b" />

## Switching the Active Namespace
By default, all kubectl commands operate in the default namespace. To permanently switch to a different namespace (e.g., dev) in your current context, run:

     kubectl config set-context $(kubectl config current-context) --namespace=dev

After setting the active namespace, executing:

    kubectl get pods

will list the Pods in the dev namespace. To view resources from other namespaces, remember to use the **--namespace** flag.


<img width="736" height="648" alt="Screenshot 2025-11-28 at 7 09 37 PM" src="https://github.com/user-attachments/assets/5d841af8-bb05-4b64-849e-2d6c542ee023" />





