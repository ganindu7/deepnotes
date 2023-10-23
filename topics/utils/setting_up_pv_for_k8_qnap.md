---
layout: default
title: setting up PV for K8 
nav_order: 2 
# permalink: /topics/utils/target_execute
parent: Utilities
---

## Installing PV 
<span style="background-color:LightGreen">
Created : 22/05/2032 | on Linux  <br />
Status: Draft
</span>

If you have already created a Storage Pool and a Volume, here are the next steps:

1. **Create an NFS Shared Folder on QNAP:**

   - In the QTS Management Console, go to "Control Panel" > "Shared Folder".
   - Click "Create" > "Shared Folder".
   - Enter a folder name and description, then choose the volume that you just created.
   - Set the disk quota for the shared folder. For instance, to make 25TB available to Kubernetes, set this as the disk quota.
   - Click "Create".

2. **Set Permissions for the NFS Shared Folder:**

   - After creating the shared folder, click on "Access Permissions" in the folder's options.
   - In the "Permission Type" dropdown, select "NFS Host Access".
   - Enter the IP address or hostname of your Kubernetes nodes. 
   - Set the "Access Right" to "No Limit".
   - Click "Apply".

3. **Enable NFS Service on QNAP:**

   - In the QTS Management Console, go to "Control Panel" > "Network & File Services" > "Win/Mac/NFS".
   - Go to the "NFS Service" tab and check "Enable NFS".
   - Click "Apply".

4. **Install NFS Client on Kubernetes Nodes:**

   Install an NFS client on each of your Kubernetes nodes. If you're using Ubuntu, you'd run:

   ```bash
   sudo apt update
   sudo apt install nfs-common
   ```

5. **Deploy NFS Provisioner to Kubernetes:**

   Deploy the NFS-client provisioner in your Kubernetes cluster. This requires Helm to be installed. If you don't have it, follow the instructions at https://helm.sh/docs/intro/install/ to install it.

   - Clone the `nfs-subdir-external-provisioner` repository:

     ```bash
     git clone https://github.com/kubernetes-sigs/nfs-subdir-external-provisioner.git
     ```

   - Deploy the NFS provisioner:

     ```bash
     cd nfs-subdir-external-provisioner/
     helm install nfs-subdir-external-provisioner ./charts/nfs-subdir-external-provisioner \
       --set nfs.server=<nfs_server_ip> \
       --set nfs.path=<path_to_your_nfs_root>
     ```
   
   Replace `<nfs_server_ip>` with the IP address of your QNAP NAS server and `<path_to_your_nfs_root>` with the path to the NFS share you created.


5.1.  **Create a default storageclass**

  
  when we create pvcs if a storage class is not specified the cluster will try to use a default storage class, the storage provisioner creates a storage class called `nfs-client` and we can make that the default storage class with the command. 

  ```
  kubectl patch storageclass nfs-client -p '{"metadata": {"annotations":{"storageclass.kubernetes.io/is-default-class":"true"}}}'
  ``` 

  with this all the new pvcs that don;t have a storage class mentioned will be allocated to the default storage class.

6. **Create PersistentVolumeClaims:**

   You can now create PersistentVolumeClaims (PVCs) in your Kubernetes cluster. These PVCs will dynamically provision PersistentVolumes (PVs) using the NFS share on your QNAP NAS.

   Here's an example PVC:

   ```yaml
   apiVersion: v1
   kind: PersistentVolumeClaim
   metadata:
     name: my-pvc
   spec:
     accessModes:
       - ReadWriteOnce
     resources:
       requests:
         storage: 1Ti
   ```

Please replace placeholders in the commands with your actual values and adjust configurations according to your environment and security requirements.



<br />

<span style="background-color:LightYellow"> Check the [**next topic**](../pytorch_walkthrough#Starting-Development-with-PyTorch)  </span>


---
*Click [here][ERRORS-SUGGESTIONS] to report Errors, make Suggestions or Comments!*

[NVIDIA-REF-1]: https://docs.nvidia.com/datacenter/cloud-native/kubernetes/install-k8s.html#option-2-installing-kubernetes-using-kubeadm
[KUBEADM-PRE-REQS]: https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/#before-you-begin
[PORTS-AND-PROTOCOLS]: https://kubernetes.io/docs/reference/networking/ports-and-protocols/
[PORT-CHK]: https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/#check-required-ports
[UBUNTU-K8-INSTALL]: https://docs.nvidia.com/datacenter/cloud-native/kubernetes/k8s-containerd.html#ubuntu-k8s
[K8]: https://kubernetes.io/
[KUBEADM-INIT]: https://kubernetes.io/docs/reference/setup-tools/kubeadm/kubeadm-init/
[NVIDIA-K8-GUIDE]: https://docs.nvidia.com/datacenter/cloud-native/kubernetes/install-k8s.html
[K8-CALICO-NETWORK-FIX]: https://forums.developer.nvidia.com/t/fix-broken-link-in-kubernets-install-pior-to-tao-api-setup/245940?u=ganindun
[K8-CONTROL-PLANE-NODE-ISOLATION-OVERRIDE]: https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/#control-plane-node-isolation
[K8-JOIN-NODES]: https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/#join-nodes


[ERRORS-SUGGESTIONS]: https://github.com/ganindu7/deepnotes/issues
[CRI-DOCKERD-RELEASE-JAN-2023]: https://github.com/Mirantis/cri-dockerd/releases/download/v0.3.1/cri-dockerd-0.3.1.amd64.tgz
[K8-SANDBOX]: https://labs.play-with-k8s.com/
[K8-CLASSROOM]: https://training.play-with-kubernetes.com/kubernetes-workshop/
[SSH-KEY-MAKING]: https://docs.github.com/en/authentication/connecting-to-github-with-ssh/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent
[ADD-SSHKEY-TO-AGENT]: https://docs.github.com/en/authentication/connecting-to-github-with-ssh/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent#adding-your-ssh-key-to-the-ssh-agent

<!-- Latex in markdown -->
<script src="https://cdn.mathjax.org/mathjax/latest/MathJax.js?config=TeX-AMS-MML_HTMLorMML" type="text/javascript"></script>
<!-- $$ \nabla_\boldsymbol{x} J(\boldsymbol{x}) $$ -->
<!-- kubectl create secret tls aisrv-gnet-secret --cert=./aisrv.gnet.lan.crt --key=./aisrv.gnet.lan.key -n default --dry-run=client -o yaml | kubectl apply -f - -->

