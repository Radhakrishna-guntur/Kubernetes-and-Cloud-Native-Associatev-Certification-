# Networking

## Networking Introduction

**Key Networking Commands**

<img width="717" height="400" alt="Screenshot 2025-12-08 at 3 49 40 PM" src="https://github.com/user-attachments/assets/6e01cce4-6faf-49d7-a47a-4c928e21defd" />



**Container Orchestration Networking**

# CNI in Kubernetes

CNI defines the responsibilities for container runtimes, and in this context, Kubernetes creates container network namespaces and links them to the appropriate network plugins. A dedicated component within Kubernetes first creates the containers and then invokes the specified CNI plugin based on the configuration.

**Kubelet Configuration for CNI**

The kubelet service on each node is the key component for configuring the CNI plugin. Within the kubelet service file, the network plugin is set to CNI and options are provided that specify the directories for both CNI plugins and configuration files. Here is an example snippet from a kubelet service file:

       ExecStart=/usr/local/bin/kubelet \
           --config=/var/lib/kubelet/kubelet-config.yaml \
           --container-runtime=remote \
           --container-runtime-endpoint=unix:///var/run/containerd/containerd.sock \
           --image-pull-progress-deadline=2m \
           --kubeconfig=/var/lib/kubelet/kubeconfig \
           --network-plugin=cni \
           --cni-bin-dir=/opt/cni/bin \
           --cni-conf-dir=/etc/cni/net.d \
           --register-node=true \
           --v=2

When you inspect the running kubelet process with a command like:

   ps -aux | grep kubelet

You will see that the network plugin is set to CNI along with additional options, such as:

The CNI binaries directory (/opt/cni/bin), which contains executables for supported plugins (e.g., bridge, DHCP, flannel).
The CNI configuration directory (/etc/cni/net.d), where the kubelet reads configuration files to determine which plugin to use.
For example, you can list the contents of these directories with the following commands:

    # Display the running kubelet process showing CNI options
     ps -aux | grep kubelet

# DNS in Kuberntes

     # List CNI plugin executables
     ls /opt/cni/bin


     # List CNI configuration files
     ls /etc/cni/net.d
