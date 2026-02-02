# Kubernetes Network Policy: Benefits, Use Cases & Best Practices:

Find out how to easily set up Network Policies that control how traffic flows within your cluster, and how the cluster interfaces with external endpoints.

Imagine you’re out in public, trying to have a conversation with a friend. At the same time, several random strangers are also shouting things at you. Not only can you not hear what your friend is trying to say, but you may also struggle to express yourself because your words are drowned out by all of the noise from other people.

This is a little like what happens in a Kubernetes cluster that lacks Network Policies. When you don’t configure a Kubernetes Network Policy for each of your pods, you may end up with lots of random network traffic flowing between pods and other endpoints. In a best case, this could degrade performance by flooding the network with unnecessary traffic. In a worse case, it may lead to security issues by making it easier for attackers who compromise one pod to spread their breach to others using the network.

The good news, though, is that – as this article explains – it’s easy enough to set up Network Policies that control how traffic flows within your cluster, as well as how the cluster interfaces with external endpoints. 

## What is a Kubernetes Network Policy?

<img width="1477" height="912" alt="image" src="https://github.com/user-attachments/assets/7ae06430-5b60-4572-ad8c-f386efac2a12" />

In Kubernetes, a Network Policy is a type of rule that determines how pods communicate over the network. Network Policies can control traffic flows between pods and other resources within a cluster. They can also manage traffic between the pods inside a cluster and external endpoints.

Let most other configuration resources in Kubernetes, Network Policies are defined using YAML code. We’ll take a look at some examples in a bit.

It’s important to note that you don’t strictly need to create a Network Policy for each pod in Kubernetes. However, without a Network Policy, Kubernetes defaults to allowing all traffic to flow into and out of a pod – which could lead to performance issues (because you may end up flooding the network with unimportant traffic and depriving workloads of sufficient bandwidth) or security challenges (due to lack of pod isolation at the network transport level).

Note also that Network Policies don’t allow management of TLS traffic encryption. To handle that, you’d use a service mesh or ingress controller.

## Key concepts for Kubernetes Network Policies
**Network Policies operate based on the following four concepts:**

**Pod selector:** The pod selector determines which pods a Kubernetes Network Policy should apply to. Selectors are typically based on pod labels; for instance, you could create a Network Policy that matches all pods labeled “database.”

**Ingress rules:** Ingress traffic rules govern which types of incoming traffic are allowed. For example, you could define a Network Policy that only allows incoming traffic from a specific range of IP addresses.

**Egress rules:** Egress rules determine which outgoing traffic a pod is allowed to send. Here again, you can allow traffic only to certain IP address ranges or only on certain ports, for example.

**Policy type:** The policy type refers to which types of traffic you want a Network Policy to manage. You can select ingress only, egress only, or both ingress and egress.

## How do Kubernetes Network Policies work?

**Network Policies work using the following steps:**

1. Define a Network Policy: First, an admin creates a Network Policy. The policy should specify which pod or pods it applies to based on pod selectors, as explained above. The policy should also define ingress and/or egress rules.
2. Apply the policy: Using kubectl, the admin applies the policy so that it takes effect.

**Automated enforcement:** Once applied, a Network Policy is automatically enforced by Kubernetes.


<img width="1477" height="912" alt="image" src="https://github.com/user-attachments/assets/b94c2952-a1f1-48fd-82b9-45fb26829a62" />


## Network Policies and CNI plugins

In the background, Network Policies rely on a Container Network Interface (CNI) plugin to manage network traffic. CNI plugins enable Kubernetes to provide networking to containers. There are a number of CNI plugins available, and all of them support basic Network Policy features like the ability to filter traffic based on IP address ranges. However, some advanced Network Policy features (like using the endPort field to specify a range of allowable networks for egress outgoing traffic) only work with certain CNI plugins.

You don’t need to specify a CNI plugin when configuring a Network Policy, but you do need a CNI plugin to be present for the policy to work (in fact, networking in Kubernetes won’t work at all unless you have a CNI plugin).

## Kubernetes Network Policy examples and use cases
To contextualize Network Policies, let’s look at some common examples and use cases.

**Enable ingress traffic only from pods in the same namespace**

The following Network Policy allows the namespace default to receive traffic only from other pods that exist within the same namespace. This may be desirable as a way of segmenting pods within a namespace at the network level.

apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-same-namespace
  namespace: default
spec:
  podSelector: {}
  policyTypes:
	- Ingress
  ingress:
	- from:
    	- podSelector: {}
The ingress: field in this policy tells Kubernetes to allow incoming traffic from all pods in the namespace that match the selector {} – which means all pods because {} matches everything.

Deny all ingress traffic to a namespace
Imagine you want to block incoming traffic to a specific namespace. This could be useful if, for example, you are using the namespace for dev/test purposes and you want to isolate it from all incoming traffic.

You can achieve this using the following Network Policy:

apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: deny-all-ingress
  namespace: dev
spec:
  podSelector: {}
  policyTypes:
	- Ingress
Here, we specify dev as the namespace to match, and the selector {} matches all pods in that namespace. The policy denies all ingress traffic because it is not configured to allow any traffic – and if a Network Policy that doesn’t explicitly allow any traffic, Kubernetes denies all traffic by default.

Allow outgoing traffic to specific external endpoints
Now, imagine you want to block outgoing network traffic from a given pod, except when the pod is communicating with specific external hosts. This could help reduce the risk of leaking sensitive data from the pod to an external endpoint.

You can achieve this using the following policy:

apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-egress-to-external
  namespace: default
spec:
  podSelector:
	matchLabels:
  	   role: my-app
  policyTypes:
	- Egress
  egress:
	- to:
    	   - ipBlock:
        	      cidr: 203.0.113.0/24
This configuration states that pods that are labeled my-app can only send egress traffic to endpoints whose IP addresses are in the 203.0.113.0/24 CIDR block.



## How to create and implement a Kubernetes Network Policy

To set up and apply a Network Policy, follow these steps.

**1. Define the policy**
First, write YAML code that defines what you want the policy to do. For example:

apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny-ingress
  namespace: default
spec:
  podSelector: {}
  policyTypes:
	- Ingress
Save the code as a file, such as my-policy.yaml.

**2. Apply the policy**
Using kubectl, apply the policy:

kubectl apply -f my-policy.yaml
You should receive a response indicating that the policy was created.

At this point, your policy is automatically implemented and in effect. Assuming the policy works as you intended, you don’t need to do anything else. 

**3. (Optional) update the Network Policy**
If the policy doesn’t work and you need to fix it, or if you simply want to make a configuration change, you can update it by modifying the YAML code, then use the kubectl apply command to apply the updated policy.

For example, imagine that, when deploying the example policy above, you accidentally specified the wrong namespace because you chose default when you meant dev. You would simply change your YAML to look like the following:

apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny-ingress
  namespace: dev
spec:
  podSelector: {}
  policyTypes:
	- Ingress
Then, save the file as my-policy.yaml and run the following command again:

kubectl apply -f my-policy.yaml

## Kubernetes Network Policies benefits
The benefits of Kubernetes Network Policies fall into two main groups – performance and security.

**Performance benefits**
From a performance perspective, Network Policies offer the following benefits:

**Bandwidth reduction:** By blocking certain types of traffic, you can eliminate unnecessary traffic. In turn, this increases network bandwidth availability and mitigates the risk of performance issues caused by poor network connectivity.
Faster troubleshooting: Because Network Policies can restrict or filter traffic, they can streamline troubleshooting in the event that a performance issue occurs. When you have a Network Policy in place, you only need to consider the networking variables allowed by the policy when trying to determine what the root cause of the problem is.
**Security benefits**
From the perspective of security, Network Policies provide these benefits:

**Network segmentation and isolation:** Network Policies let you segment workloads from each other and from external endpoints. In this way, they can block malicious traffic and help prevent Denial-of-Service (DoS) attacks.‍
Compliance: In some cases, compliance rules may mandate that you keep certain types of protected data out of a specific geographic region or system. Network Policies help here by allowing you to restrict where traffic can flow. For example, you could block egress from a pod to endpoints in a certain region.

<img width="1477" height="912" alt="image" src="https://github.com/user-attachments/assets/0b5c951e-67b5-4b9b-aa48-dfabfb713b2f" />

## Challenges in implementing Kubernetes Network Policies

**Network policies are a powerful tool, but they can be difficult to implement. Common challenges include:**

**Policy complexity**: Having to configure different rules for each policy can be a complex task, especially in a large cluster that includes dozens of different policies.
Policy conflicts: You can end up with conflicts between Network Policies in the case where you create more than one policy that matches the same pod and defines different rules.
Performance overhead: Network Policies create performance overhead because Kubernetes consumes resources when evaluating and enforcing the policies. The overhead for each policy is typically small, but if you have a large number of policies, or if your policies include many rules, the overhead can become significant, leaving fewer resources for your workloads.
Network Policy best practices
To mitigate the risk of Network Policy issues and make your policies as effective as possible, consider the following best practices.

**Create a default policy for your entire cluster**
Since pods not covered by a Network Policy have no protections at the network level, it’s a best practice to ensure that all pods throughout your cluster are covered by a default policy.

The easiest way to do this is to create a default policy with no namespace specified and with {} as the podSelector value. This will match all pods in all namespaces. You can include explicit deny policies in this policy to block all traffic by default, then create additional policies that grant access on an as-needed basis to specific namespaces or pods.

**Start with namespace-level policies**
In general, managing networking rules based on namespaces rather than individual pods is simpler and better from a resource perspective because it leaves you with fewer policies to configure. Configure policies on a per-pod basis only when you require a level of granularity that you can’t achieve using namespace-level policies.

Use policy types that support ingress and egress
You can also reduce total policy count (and, by extension, complexity) by making each Network Policy support both ingress and egress. This eliminates the need to create separate incoming and outgoing traffic policies.

**Avoid policy conflicts**
Where possible, avoid creating more than one policy that matches the same pod, since this could lead to conflicts. Having multiple policies for a given pod may be unavoidable in cases where the pod includes multiple labels that match different policies. But in general, this is unlikely to be necessary because it’s not common for the same workload to require different types of networking rules.

**Know your CNI**
Since Network Policy behavior and features depend to some extent on the CNI enabled in your cluster, it’s important to know which CNI you are using.

The default CNIs that ship with Kubernetes vary by distribution, so check your distribution’s documentation if you’re unsure which CNI you are using. Alternatively, check the /etc/cni/net.d/ directory on a control plane node; this is usually where configuration files for your cluster’s CNI reside, so you can look at those files to tell which CNI (or CNIs) are in place.

## Best practice	Description
Create a default policy for your entire cluster	A default policy ensures that all pods are covered by networking rules.

Use namespace-level policies	Unless you require the granularity of pod-level policies, configure policies to operate at the namespace level. This reduces complexity.
Use ingress+egress policy types	Policies that define both ingress and egress rules in a single file reduce the number of policies you need to manage.
Avoid policy conflicts	Aim to avoid defining more than one policy that matches the same pod.

Know your CNI	Knowing which CNI network plugin you're using is important because some policy features work only with certain CNIs.


## Network Policy troubleshooting tips

If a policy is not working as expected, the following steps can help you determine the root cause of the issue:

**Verify CNI configuration:** Start by checking your documentation or the /etc/cni/net.d/ directory to determine which CNI your cluster is using. Then, look at the CNI’s network policy documentation to determine whether there are any limitations, such as a lack of support for a certain feature enabled in one or more of your policies.
**Check pod labels and selectors:** Reviewing pod labels and selectors may identify misconfigurations, such as a typo that causes a Network Policy not to apply to the right pods.
**Look for policy conflicts:** Check whether multiple policies apply to the same pod and if so, whether they conflict in any way.
**Check log files:** Most CNIs log information about network behavior, and this information may help troubleshoot Network Policy issues. Unfortunately, the location of log files varies between CNIs, so you’ll need to check your CNI’s documentation to determine where to find the logs.

## Kubernetes Network Policy tools and integrations

When working with Network Policies, the following tools can help streamline the deployment and troubleshooting process:

**CNI plugins:** As we mentioned, CNI network plugins are the backbone of Network Policies. From a Network Policy troubleshooting perspective, CNI plugins are especially valuable as a means of logging information – which is important because Kubernetes doesn’t directly log Network Policy data on its own.

**Open source tools:** Open source tools, such YAML Validator, can help check for syntax issues in Network Policy configurations.

**eBPF:** The extended Berkeley Packet Filter, or eBPF, is a powerful tool that can monitor traffic as it flows across pods and nodes. Using eBPF observability, you can gain deep visibility into where packers are moving and what is causing delays – which in turn can help you to troubleshoot networking issues whose root causes are not obvious based on log files or other data within your cluster.

## How groundcover enhances Kubernetes Network Policy management

We just mentioned eBPF, but we’ll mention it again, because it’s the secret sauce that makes groundcover so powerful in the realm of Kubernetes Network Policy management and troubleshooting. Under the hood, groundcover uses eBPF in Kubernetes to collect low-level data about Kubernetes network performance – which means you can quickly get to the root of networking issues.

If you want to know, for instance, which packets are flowing to and from which pods, and how those flows align with network policies, groundcover can tell you quickly – without requiring you to know details about which CNI network plugin you’re using or sort through CNI-specific log files.

## Making a policy of healthy Network Policies
Like most things in Kubernetes, Network Policies are powerful but complex. You’ll want to set up Network Policies for all but the most simple of clusters, yet you may also find that your policies don’t always work as you expect – which is why the ability to troubleshoot network behavior with help from tools like groundcover is critical for making the most of the Kubernetes Network Policy feature.


Reference: https://www.groundcover.com/blog/kubernetes-network-policy
