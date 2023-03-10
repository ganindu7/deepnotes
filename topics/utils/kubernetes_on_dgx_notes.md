---
layout: default
title: Installing Kubernetes on Nvidia DGX 
nav_order: 2 
# permalink: /topics/utils/target_execute
parent: Utilities
---

## Installing Kubernetes on DGX using Kubeadm [PyTorch][NVIDIA-REF-1]
<span style="background-color:LightGreen">
Created : 09/03/2023 | on Linux dgx 5.4.0-144-generic #161-Ubuntu SMP Fri Feb 3 14:49:04 UTC 2023 x86_64 x86_64 x86_64 GNU/Linux <br />
Updated : 09/03/2032 | on Linux dgx 5.4.0-144-generic #161-Ubuntu SMP Fri Feb 3 14:49:04 UTC 2023 x86_64 x86_64 x86_64 GNU/Linux <br />
Status: Draft
</span>


### Writeup

"A [kubernetes][K8] cluster is composed of master nodes and worker nodes. the master nodes run the control plane components". these components include 

* API server (front end of the kubectl CLI),
* `etcd` (stores the cluster state and the others)
* Scheduler
* Controller Manager 

*Control plane components can have some impact on CPU intensive tasks and conversely, CPU or HDD/SSD intensive tasks can have a high impact on your control plane components.*  

Ideally use CPU-only (GPU Free) master nodes to run the control plane components: 

# [`kubeadm` pre-requisites][KUBEADM-PRE-REQS]

* Check network adapters and [ports][PORTS-AND-PROTOCOLS] for their availability (if your master and slave nodes are in different network segments firewall attributes too). (e.g. [port check][PORT-CHK]
* Disable swap on the nodes so kubelet can work correctly
* container runtime like Docker, containerd or CRI-O

for ubuntu follow [this][UBUNTU-K8-INSTALL] to install k8. (*upto initialising kubeadm with `kubeadm init`*)

* If you are using dgx, docker is pre installed so I think it makes sense to use it rather than installing a different CRI, to use Docker's CRI head to [cri-dockerd repo][https://github.com/Mirantis/cri-dockerd] and follow the instructions or download [pre-built-binaries][CRI-DOCKERD-RELEASE-JAN-2023](this one is for amd64 released on January 2023)

<span style="background-color:LightYellow">
Note: when initialising `kubeadm` use the correct IP-ranges (e.g. `192.168.0.0/16` or `172.16.1.0/24`, I used `172.16.5.0/28` so I can have $$2^4  - 2= 14$$ (omitting the two end addresses) host addresses `(240-254)` )
</span>

Refer to [`kubeadm init` page][KUBEADM-INIT] for more info. 


* initialise kubeadm (I am using cri-dockerd and the network config `172.16.5.0/24`)

```
sudo kubeadm init --pod-network-cidr=172.16.5.0/24 --cri-socket=unix:///var/run/cri-dockerd.sock
```

You should get something like below (this is what I got)

```
[init] Using Kubernetes version: v1.26.2
[preflight] Running pre-flight checks
[preflight] Pulling images required for setting up a Kubernetes cluster
[preflight] This might take a minute or two, depending on the speed of your internet connection
[preflight] You can also perform this action in beforehand using 'kubeadm config images pull'
[certs] Using certificateDir folder "/etc/kubernetes/pki"
[certs] Generating "ca" certificate and key
[certs] Generating "apiserver" certificate and key
[certs] apiserver serving cert is signed for DNS names [gsrv kubernetes kubernetes.default kubernetes.default.svc kubernetes.default.svc.cluster.local] and IPs [10.96.0.1 172.16.1.19]
[certs] Generating "apiserver-kubelet-client" certificate and key
[certs] Generating "front-proxy-ca" certificate and key
[certs] Generating "front-proxy-client" certificate and key
[certs] Generating "etcd/ca" certificate and key
[certs] Generating "etcd/server" certificate and key
[certs] etcd/server serving cert is signed for DNS names [gsrv localhost] and IPs [172.16.1.19 127.0.0.1 ::1]
[certs] Generating "etcd/peer" certificate and key
[certs] etcd/peer serving cert is signed for DNS names [gsrv localhost] and IPs [172.16.1.19 127.0.0.1 ::1]
[certs] Generating "etcd/healthcheck-client" certificate and key
[certs] Generating "apiserver-etcd-client" certificate and key
[certs] Generating "sa" key and public key
[kubeconfig] Using kubeconfig folder "/etc/kubernetes"
[kubeconfig] Writing "admin.conf" kubeconfig file
[kubeconfig] Writing "kubelet.conf" kubeconfig file
[kubeconfig] Writing "controller-manager.conf" kubeconfig file
[kubeconfig] Writing "scheduler.conf" kubeconfig file
[kubelet-start] Writing kubelet environment file with flags to file "/var/lib/kubelet/kubeadm-flags.env"
[kubelet-start] Writing kubelet configuration to file "/var/lib/kubelet/config.yaml"
[kubelet-start] Starting the kubelet
[control-plane] Using manifest folder "/etc/kubernetes/manifests"
[control-plane] Creating static Pod manifest for "kube-apiserver"
[control-plane] Creating static Pod manifest for "kube-controller-manager"
[control-plane] Creating static Pod manifest for "kube-scheduler"
[etcd] Creating static Pod manifest for local etcd in "/etc/kubernetes/manifests"
[wait-control-plane] Waiting for the kubelet to boot up the control plane as static Pods from directory "/etc/kubernetes/manifests". This can take up to 4m0s
[apiclient] All control plane components are healthy after 22.503248 seconds
[upload-config] Storing the configuration used in ConfigMap "kubeadm-config" in the "kube-system" Namespace
[kubelet] Creating a ConfigMap "kubelet-config" in namespace kube-system with the configuration for the kubelets in the cluster
[upload-certs] Skipping phase. Please see --upload-certs
[mark-control-plane] Marking the node gsrv as control-plane by adding the labels: [node-role.kubernetes.io/control-plane node.kubernetes.io/exclude-from-external-load-balancers]
[mark-control-plane] Marking the node gsrv as control-plane by adding the taints [node-role.kubernetes.io/control-plane:NoSchedule]
[bootstrap-token] Using token: ju10gj.8mu5ziozryur4dnc
[bootstrap-token] Configuring bootstrap tokens, cluster-info ConfigMap, RBAC Roles
[bootstrap-token] Configured RBAC rules to allow Node Bootstrap tokens to get nodes
[bootstrap-token] Configured RBAC rules to allow Node Bootstrap tokens to post CSRs in order for nodes to get long term certificate credentials
[bootstrap-token] Configured RBAC rules to allow the csrapprover controller automatically approve CSRs from a Node Bootstrap Token
[bootstrap-token] Configured RBAC rules to allow certificate rotation for all node client certificates in the cluster
[bootstrap-token] Creating the "cluster-info" ConfigMap in the "kube-public" namespace
[kubelet-finalize] Updating "/etc/kubernetes/kubelet.conf" to point to a rotatable kubelet client certificate and key
[addons] Applied essential addon: CoreDNS
[addons] Applied essential addon: kube-proxy

Your Kubernetes control-plane has initialized successfully!

To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

Alternatively, if you are the root user, you can run:

  export KUBECONFIG=/etc/kubernetes/admin.conf

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/

Then you can join any number of worker nodes by running the following on each as root:

kubeadm join 172.16.1.19:6443 --token <a_token> \
	--discovery-token-ca-cert-hash sha256:<a_unique_hash_string> 

```

<br/>

* If you have an issue with multiple CRI endpoints please refer to this [stackoverflow answer](https://stackoverflow.com/a/72878730)
<br/>
* You can find more info in installing kubectl [here](https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/)
<br/>
* If you want to verify the install check here [here](https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/#verify-kubectl-configuration)
<br/><br/>


Pulling config images (I don't fully understand this yet, just following the prompt *hopefully will update as I learn more*)

```
kubeadm config image pull
```
Output:
```
[config/images] Pulled registry.k8s.io/kube-apiserver:v1.26.2
[config/images] Pulled registry.k8s.io/kube-controller-manager:v1.26.2
[config/images] Pulled registry.k8s.io/kube-scheduler:v1.26.2
[config/images] Pulled registry.k8s.io/kube-proxy:v1.26.2
[config/images] Pulled registry.k8s.io/pause:3.9
[config/images] Pulled registry.k8s.io/etcd:3.5.6-0
[config/images] Pulled registry.k8s.io/coredns/coredns:v1.9.3

```

Now we try: `kubectl get pods -A`

output:
```
NAMESPACE     NAME                           READY   STATUS    RESTARTS   AGE
kube-system   coredns-787d4945fb-46m9v       0/1     Pending   0          137m
kube-system   coredns-787d4945fb-bf82h       0/1     Pending   0          137m
kube-system   etcd-gsrv                      1/1     Running   0          137m
kube-system   kube-apiserver-gsrv            1/1     Running   0          137m
kube-system   kube-controller-manager-gsrv   1/1     Running   0          137m
kube-system   kube-proxy-k95h8               1/1     Running   0          137m
kube-system   kube-scheduler-gsrv            1/1     Running   0          137m
```

<span style="background-color:orange;">
Note: 
judging by the output of this command aprt from the `"coredns-"*` everything seems to be in the `"RUNNINNG"` state. I belive at this point I can get things working with numerical hostnames insteads of strings. 
</span>


because of the following lines  in the `kubeadm init ` output from earlier I think the next step is to get some worker node(s) running. 


```
Then you can join any number of worker nodes by running the following on each as root:

kubeadm join 172.16.1.19:6443 --token <a_token> \
	--discovery-token-ca-cert-hash sha256:<a_unique_hash_string> 

```

### [HELM](https://helm.sh/)

Now we install [HELM](https://helm.sh/) (a package manager for Kubernetes)

you can install helm in many ways I used the [script method](https://helm.sh/docs/intro/install/#from-script)

Once Helm is in place you can do a [quick tutorial](https://helm.sh/docs/intro/quickstart/#initialize-a-helm-chart-repository) to understand how it works.




I think I should be able to make API calls to the master node and get them relayed to worker nodes when the worker nodes are initialised(or `join`ed with the token)


### joining a worker node {this is my current plan but I'm not sure if this is what nvidia recommends or if there is a nvidia specific way --> waiting response from nvidia}:

<br/>


first we need to get a join token from the master node, to get a join token run the following command in the master node.  

```
kubeadm token create --print-join-command
```

To join  DGX Station A100 as a worker node to a Kubernetes cluster, you will need to install the following software and dependencies on the DGX Station:

* Docker: Kubernetes requires a container runtime to run containerized workloads. Docker is a popular choice for container runtime, and it is compatible with Kubernetes. You can install Docker on your DGX Station by following the instructions on the Docker website.

* kubelet: kubelet is a Kubernetes agent that runs on each worker node and communicates with the Kubernetes master node. You can install kubelet on your DGX Station by following the instructions provided in the official Kubernetes documentation for your specific operating system and version.

* kubeadm: kubeadm is a command-line tool used to bootstrap a Kubernetes cluster. You will need to install kubeadm on the DGX Station to join it to the cluster.

* kubectl: kubectl is a command-line tool used to interact with a Kubernetes cluster. You will need to install kubectl on the DGX Station to manage the cluster.


To install I suggest to follow the instructions [here](https://docs.nvidia.com/datacenter/cloud-native/kubernetes/k8s-containerd.html#install-kubernetes-components)


Once you have installed these software dependencies on the DGX Station, 

Obtain the join token from the Kubernetes master node by running the following command on the master node:

```
kubeadm token create --print-join-command
```
Copy the output of the above command, which will be a kubeadm join command with a token and the IP address of the master node.

SSH into the worker node and run the kubeadm join command obtained in step 2. This will join the worker node to the Kubernetes cluster.

Verify that the worker node has joined the cluster by running the following command on the master node:

```
kubectl get nodes
```
The output of this command should list all the nodes in the cluster, including the new worker node.

Optional: Label the worker node to make it easier to schedule pods on it. For example, you could label the node as "worker" by running the following command:

```
kubectl label node <worker-node-name> node-role.kubernetes.io/worker=worker
```










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

[ERRORS-SUGGESTIONS]: https://github.com/ganindu7/deepnotes/issues
[CRI-DOCKERD-RELEASE-JAN-2023]: https://github.com/Mirantis/cri-dockerd/releases/download/v0.3.1/cri-dockerd-0.3.1.amd64.tgz

<!-- Latex in markdown -->
<script src="https://cdn.mathjax.org/mathjax/latest/MathJax.js?config=TeX-AMS-MML_HTMLorMML" type="text/javascript"></script>
<!-- $$ \nabla_\boldsymbol{x} J(\boldsymbol{x}) $$ -->