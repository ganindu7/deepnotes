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

This document is an adaptation of the [nvidia-guide][NVIDIA-K8-GUIDE] for datacenter, k8 setup.

### Control plane components 

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


### GPU Nodes

At this point I think we have a working Kubernetes control plane 


### Nvidia Device plugin for Kubernetes 

Kubernetes provide a device plugin framework that you can use to advertise system hardware resources to the Kubelet. With this you or other hardware vendors such as nvidia can implements device plugins that can be either installed manually or as a `DaemonSet`. DGX is an example for one of these devices. 

Kubernetes also provides an [operator framework](https://cloud.redhat.com/blog/introducing-the-operator-framework) to help package, deploy and mange applications k8 applications. the operator framework is essentially an extension of the support structure that is usually necessary for an application to operate in a working environment (i.e. hardware, OS, drivers, telemetry and health/services and management)


[ngc catalog link](https://catalog.ngc.nvidia.com/orgs/nvidia/containers/k8s-device-plugin)
[nvidia-gpu operator](https://github.com/NVIDIA/gpu-operator)

<br/>


I think I should be able to make API calls to the master node and get them relayed to worker nodes when the worker nodes are initialised(or `join`ed with the token)

enable cri_plugin in DGX

```
/etc/containerd/config.toml 

# disabled_plugins = ["cri"]

#root = "/var/lib/containerd"
#state = "/run/containerd"
#subreaper = true
#oom_score = 0

#[grpc]
#  address = "/run/containerd/containerd.sock"
#  uid = 0
#  gid = 0

#[debug]
#  address = "/run/containerd/debug.sock"
#  uid = 0
#  gid = 0
#  level = "info"                
```
Using the token I created earlier (after `sudo systemctl restart containerd` in the DGX an)

```
sudo kubeadm join [IP]:6443 --token <token> --discovery-token-ca-cert-hash <hash> 
```

output:

```
[preflight] Running pre-flight checks
[preflight] Reading configuration from the cluster...
[preflight] FYI: You can look at this config file with 'kubectl -n kube-system get cm kubeadm-config -o yaml'
[kubelet-start] Writing kubelet configuration to file "/var/lib/kubelet/config.yaml"
[kubelet-start] Writing kubelet environment file with flags to file "/var/lib/kubelet/kubeadm-flags.env"
[kubelet-start] Starting the kubelet
[kubelet-start] Waiting for the kubelet to perform the TLS Bootstrap...

This node has joined the cluster:
* Certificate signing request was sent to apiserver and a response was received.
* The Kubelet was informed of the new secure connection details.

Run 'kubectl get nodes' on the control-plane to see this node join the cluster.

```

Now we can test for the node visibility on the control plane with 

```
kubectl get nodes
```

I ran the code above to get the following output 

```
NAME   STATUS     ROLES           AGE   VERSION
dgx    NotReady   <none>          60m   v1.26.2
gsrv   NotReady   control-plane   8h    v1.26.2

```

here dgx is the dgx station and the gsrv is the CPU server. 


Then install the nvidia HELM REPO 

```
helm repo add nvidia https://helm.ngc.nvidia.com/nvidia \
   && helm repo update
```

I labelled the dgx node with (ran the command in the master node)

```
 kubectl label node dgx node-role.kubernetes.io/worker=worker
```


## What is next (awaiting support from nvidia)

1. Do Ihave to install the nvidia container toolkit on DGX (is is it already installed)
2. Do I have to Install https://github.com/NVIDIA/k8s-device-plugin#deployment-via-helm in the control plane or the master node 

The main problem seems to be the network.

My end goal is to get [this](https://docs.nvidia.com/tao/tao-toolkit/text/tao_toolkit_api/api_overview.html)(TAO toolkit with the REST API via k8) working.


Note: I tried installing calico from it's web [instructions](https://docs.tigera.io/calico/3.25/getting-started/kubernetes/self-managed-onprem/onpremises) but didn't see any changes I still get 

Note: My Ansible version from the cpu server which acts as the master node is `Ubuntu 22.04.1 LTS`

hence when I run `ansible localhost -m setup -a 'filter=ansible_distribution_version'` from it I get

```
localhost | SUCCESS => {
    "ansible_facts": {
        "ansible_distribution_version": "22.04"
    },
    "changed": false
}

```

this leads to 

```
TASK [check os version] ***********************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************
fatal: [172.16.1.19]: FAILED! => {
    "assertion": "ansible_distribution_version in ['18.04', '20.04']",
    "changed": false,
    "evaluated_to": false,
    "msg": "Assertion failed"
}

```

when I run. [this instruction](https://docs.nvidia.com/tao/tao-toolkit/text/tao_toolkit_api/api_setup.html#:~:text=cnc/cnc_values_3.1.yaml-,Proceed%20with%20deployment.,-bash%20setup.sh)








```

g@gsrv:~/k8$ kubectl get nodes
NAME   STATUS     ROLES           AGE    VERSION
dgx    NotReady   worker          164m   v1.26.2
gsrv   NotReady   control-plane   10h    v1.26.2
g@gsrv:~/k8$ 


```

when I get more info with describe node I get 

```
g@gsrv:~/k8$ kubectl describe node dgx
Name:               dgx
Roles:              worker
Labels:             beta.kubernetes.io/arch=amd64
                    beta.kubernetes.io/os=linux
                    kubernetes.io/arch=amd64
                    kubernetes.io/hostname=dgx
                    kubernetes.io/os=linux
                    node-role.kubernetes.io/worker=worker
Annotations:        kubeadm.alpha.kubernetes.io/cri-socket: unix:///var/run/containerd/containerd.sock
                    node.alpha.kubernetes.io/ttl: 0
                    volumes.kubernetes.io/controller-managed-attach-detach: true
CreationTimestamp:  Fri, 10 Mar 2023 17:27:08 +0000
Taints:             node.kubernetes.io/not-ready:NoSchedule
Unschedulable:      false
Lease:
  HolderIdentity:  dgx
  AcquireTime:     <unset>
  RenewTime:       Fri, 10 Mar 2023 20:14:16 +0000
Conditions:
  Type             Status  LastHeartbeatTime                 LastTransitionTime                Reason                       Message
  ----             ------  -----------------                 ------------------                ------                       -------
  MemoryPressure   False   Fri, 10 Mar 2023 20:11:34 +0000   Fri, 10 Mar 2023 17:27:08 +0000   KubeletHasSufficientMemory   kubelet has sufficient memory available
  DiskPressure     False   Fri, 10 Mar 2023 20:11:34 +0000   Fri, 10 Mar 2023 17:27:08 +0000   KubeletHasNoDiskPressure     kubelet has no disk pressure
  PIDPressure      False   Fri, 10 Mar 2023 20:11:34 +0000   Fri, 10 Mar 2023 17:27:08 +0000   KubeletHasSufficientPID      kubelet has sufficient PID available
  Ready            False   Fri, 10 Mar 2023 20:11:34 +0000   Fri, 10 Mar 2023 17:27:08 +0000   KubeletNotReady              container runtime network not ready: NetworkReady=false reason:NetworkPluginNotReady message:Network plugin returns error: cni plugin not initialized
Addresses:
  InternalIP:  172.16.3.2
  Hostname:    dgx
Capacity:
  cpu:                128
  ephemeral-storage:  1843269236Ki
  hugepages-1Gi:      0
  hugepages-2Mi:      0
  memory:             528018224Ki
  pods:               110
Allocatable:
  cpu:                128
  ephemeral-storage:  1698756925085
  hugepages-1Gi:      0
  hugepages-2Mi:      0
  memory:             527915824Ki
  pods:               110
System Info:
  Machine ID:                 fe86cb15be594307a11cc9847b0eb5c2
  System UUID:                21af0608-1dd2-11b2-9c02-f24e4f55ad5c
  Boot ID:                    0f544fd5-41e8-4f48-b326-0c58d2e99fb9
  Kernel Version:             5.4.0-144-generic
  OS Image:                   Ubuntu 20.04.5 LTS
  Operating System:           linux
  Architecture:               amd64
  Container Runtime Version:  containerd://1.6.10
  Kubelet Version:            v1.26.2
  Kube-Proxy Version:         v1.26.2
Non-terminated Pods:          (2 in total)
  Namespace                   Name                                CPU Requests  CPU Limits  Memory Requests  Memory Limits  Age
  ---------                   ----                                ------------  ----------  ---------------  -------------  ---
  kube-system                 kube-proxy-gfkkr                    0 (0%)        0 (0%)      0 (0%)           0 (0%)         167m
  tigera-operator             tigera-operator-54b47459dd-qdwtt    0 (0%)        0 (0%)      0 (0%)           0 (0%)         28m
Allocated resources:
  (Total limits may be over 100 percent, i.e., overcommitted.)
  Resource           Requests  Limits
  --------           --------  ------
  cpu                0 (0%)    0 (0%)
  memory             0 (0%)    0 (0%)
  ephemeral-storage  0 (0%)    0 (0%)
  hugepages-1Gi      0 (0%)    0 (0%)
  hugepages-2Mi      0 (0%)    0 (0%)
Events:
  Type    Reason            Age                    From           Message
  ----    ------            ----                   ----           -------
  Normal  CIDRNotAvailable  2m48s (x39 over 167m)  cidrAllocator  Node dgx status is now: CIDRNotAvailable
g@gsrv:~/k8$ 

```


## Resetting DGX 

```
sudo kubeadm reset
W0313 15:23:17.052577   96842 preflight.go:56] [reset] WARNING: Changes made to this host by 'kubeadm init' or 'kubeadm join' will be reverted.
[reset] Are you sure you want to proceed? [y/N]: y
[preflight] Running pre-flight checks
W0313 15:23:18.416975   96842 removeetcdmember.go:106] [reset] No kubeadm config, using etcd pod spec to get data directory
[reset] Deleted contents of the etcd data directory: /var/lib/etcd
[reset] Stopping the kubelet service
[reset] Unmounting mounted directories in "/var/lib/kubelet"
[reset] Deleting contents of directories: [/etc/kubernetes/manifests /var/lib/kubelet /etc/kubernetes/pki]
[reset] Deleting files: [/etc/kubernetes/admin.conf /etc/kubernetes/kubelet.conf /etc/kubernetes/bootstrap-kubelet.conf /etc/kubernetes/controller-manager.conf /etc/kubernetes/scheduler.conf]

The reset process does not clean CNI configuration. To do so, you must remove /etc/cni/net.d

The reset process does not reset or clean up iptables rules or IPVS tables.
If you wish to reset iptables, you must do so manually by using the "iptables" command.

If your cluster was setup to utilize IPVS, run ipvsadm --clear (or similar)
to reset your system's IPVS tables.

The reset process does not clean your kubeconfig files and you must remove them manually.
Please, check the contents of the $HOME/.kube/config file.

```

### Setting up DGX as both the master and the GPU worker node 

After learning that my CPU server running on ubuntu 22.04 is not compatible with current TAO K8 setup I had to accept defeat and switch to the original guide 
therefore reverting back to this [guide][NVIDIA-K8-GUIDE]


* Disable swap 

```
sudo swapoff -a
```

* Initialise as a master (note i've used a different network segment)

```
 sudo kubeadm init --pod-network-cidr=172.16.5.0/24

```

output:

```
[init] Using Kubernetes version: v1.26.2
[preflight] Running pre-flight checks
[preflight] Pulling images required for setting up a Kubernetes cluster
[preflight] This might take a minute or two, depending on the speed of your internet connection
[preflight] You can also perform this action in beforehand using 'kubeadm config images pull'
[certs] Using certificateDir folder "/etc/kubernetes/pki"
[certs] Generating "ca" certificate and key
[certs] Generating "apiserver" certificate and key
[certs] apiserver serving cert is signed for DNS names [dgx kubernetes kubernetes.default kubernetes.default.svc kubernetes.default.svc.cluster.local] and IPs [10.96.0.1 172.16.3.2]
[certs] Generating "apiserver-kubelet-client" certificate and key
[certs] Generating "front-proxy-ca" certificate and key
[certs] Generating "front-proxy-client" certificate and key
[certs] Generating "etcd/ca" certificate and key
[certs] Generating "etcd/server" certificate and key
[certs] etcd/server serving cert is signed for DNS names [dgx localhost] and IPs [172.16.3.2 127.0.0.1 ::1]
[certs] Generating "etcd/peer" certificate and key
[certs] etcd/peer serving cert is signed for DNS names [dgx localhost] and IPs [172.16.3.2 127.0.0.1 ::1]
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
[apiclient] All control plane components are healthy after 4.501125 seconds
[upload-config] Storing the configuration used in ConfigMap "kubeadm-config" in the "kube-system" Namespace
[kubelet] Creating a ConfigMap "kubelet-config" in namespace kube-system with the configuration for the kubelets in the cluster
[upload-certs] Skipping phase. Please see --upload-certs
[mark-control-plane] Marking the node dgx as control-plane by adding the labels: [node-role.kubernetes.io/control-plane node.kubernetes.io/exclude-from-external-load-balancers]
[mark-control-plane] Marking the node dgx as control-plane by adding the taints [node-role.kubernetes.io/control-plane:NoSchedule]
[bootstrap-token] Using token: hj5i5l.b03mf63sxolhn28n
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

kubeadm join 172.16.3.2:6443 --token <a.token> \
  --discovery-token-ca-cert-hash sha256:<a_hash> 

```

Note: The tokens will expire after some time use `kubeadm token list` to check the current valid tokens

Then ran these commands as requested 

```
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config

```

#### Network stuff 

The suggested command fails 

```
kubectl apply -f https://docs.projectcalico.org/manifests/calico.yaml
```

Try this [fix][K8-CALICO-NETWORK-FIX] suggest by me. 

#### untaint the control plane 


This is only so the DGX (now a master node because we ran `kubeadm init` on that) can be used to shedule GPU pods.

```
kubectl taint nodes --all node-role.kubernetes.io/master-

```  

again! this command also fails 

so try [changing the default control plane node as suggested by this][K8-CONTROL-PLANE-NODE-ISOLATION-OVERRIDE]

```
kubectl taint nodes --all node-role.kubernetes.io/control-plane-
```

Now you may be able to list join tokens with

```
kubeadm token list

```

create new tokens with 

```
kubeadm token create

```

more info under [this topic][K8-JOIN_NODES]


### TAO API SETUP





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

<!-- Latex in markdown -->
<script src="https://cdn.mathjax.org/mathjax/latest/MathJax.js?config=TeX-AMS-MML_HTMLorMML" type="text/javascript"></script>
<!-- $$ \nabla_\boldsymbol{x} J(\boldsymbol{x}) $$ -->