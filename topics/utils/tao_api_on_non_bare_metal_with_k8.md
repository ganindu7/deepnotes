---
layout: default
title: TAO API with Kubernetes in non bare metal 
# nav_order: 2 
# permalink: /topics/utils/target_execute
parent: Utilities
---

## Setting up TAO API with Kubernetes. 

For this to work a separate k8 master node is needed. The cluster we are building consists of a master node (CPU server) and a GPU node (for our example the GPU node is an NVIDIA DGX system).


### Step 0 [to be run on Master node ]


To verify you can run `kubectl get nodes` on your master. 

If the GPU node is already registered (maybe from previous attempts) you can remove or reset (in an initial setup you can run `sudo kubeadm reset` with the appropriate arguments)

after reset 

```
sudo kubeadm reset --cri-socket=unix:///var/run/cri-dockerd.sock


rm /etc/cni/net.d
rm $HOME/.kube

sudo iptables -P INPUT ACCEPT
sudo iptables -P FORWARD ACCEPT
sudo iptables -P OUTPUT ACCEPT
sudo iptables -t nat -F
sudo iptables -t mangle -F
sudo iptables -F
sudo iptables -X

```

Then fresh setup:

```
sudo kubeadm init --pod-network-cidr=192.168.0.0/16 --cri-socket=unix:///var/run/cri-dockerd.sock

```

Then run below commands to set up kube config

```
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config

```

install and configure network operator 

```
kubectl create -f https://raw.githubusercontent.com/projectcalico/calico/v3.25.0/manifests/tigera-operator.yaml
```

download to a custom location if needed 

```
curl https://raw.githubusercontent.com/projectcalico/calico/v3.25.0/manifests/custom-resources.yaml -O
kubectl create -f custom-resources.yaml

```

Install calicoctl binary (if needed)

```
curl -L https://github.com/projectcalico/calico/releases/latest/download/calicoctl-linux-amd64 -o kubectl-calico

chmod +x kubectl-calico
```

install calicoctl as a kubectl plugin

```
curl -L https://github.com/projectcalico/calico/releases/latest/download/calicoctl-linux-amd64 -o kubectl-calico

kubectl calico -h


```

now you can see the IP pool with `kubectl calico ipam show`

```
kubectl calico ipam show

+----------+----------------+-----------+------------+--------------+
| GROUPING |      CIDR      | IPS TOTAL | IPS IN USE |   IPS FREE   |
+----------+----------------+-----------+------------+--------------+
| IP Pool  | 192.168.0.0/16 |     65536 | 7 (0%)     | 65529 (100%) |
+----------+----------------+-----------+------------+--------------+

```

then verify the master node with 

```
kubectl get nodes
```
Then verify pods with 

```
kubectl get pods -A
NAMESPACE          NAME                                       READY   STATUS    RESTARTS   AGE
calico-apiserver   calico-apiserver-685597b56-fltsj           1/1     Running   0          33m
calico-apiserver   calico-apiserver-685597b56-kl5rt           1/1     Running   0          33m
calico-system      calico-kube-controllers-6b7b9c649d-hsfw8   1/1     Running   0          35m
calico-system      calico-node-28wfp                          1/1     Running   0          35m
calico-system      calico-typha-7cb999b6dd-fxvps              1/1     Running   0          35m
calico-system      csi-node-driver-4l4zk                      2/2     Running   0          34m
kube-system        coredns-787d4945fb-7zj7r                   1/1     Running   0          50m
kube-system        coredns-787d4945fb-p5rsr                   1/1     Running   0          50m
kube-system        etcd-gsrv                                  1/1     Running   0          50m
kube-system        kube-apiserver-gsrv                        1/1     Running   0          50m
kube-system        kube-controller-manager-gsrv               1/1     Running   0          50m
kube-system        kube-proxy-fqrwh                           1/1     Running   0          50m
kube-system        kube-scheduler-gsrv                        1/1     Running   0          50m
tigera-operator    tigera-operator-54b47459dd-6bpx5           1/1     Running   0          35m
```


get a join command for the GPU node 

```
kubeadm token create --print-join-command

```

NOTE: After adding the node you can label it as a gpuworker

```
kubectl label node dgx node-role.kubernetes.io/gpuworker=gpuworker --overwrite=True
```

On the gpu node (DGX)
if  `/etc/docker/daemon.json` is not as following 

```
{
    "runtimes": {
        "nvidia": {
            "path": "nvidia-container-runtime",
            "runtimeArgs": []
        }
    }
}

```

on the dgx install nvidia-docker 

```
 sudo apt install nvidia-docker2
 sudo systemctl restart docker

```

### Step 1 [to be run on GPU node ]
* [Install Kubernetes][NVIDIA-INSTALL-K8] 
* [install nvidia GPU operator][INSTALL-NVIDIA-GPU-OPERATOR]

Add the node to the cluster we made earlier with the generated join command. 

### Step 3 [to be run on the CPU Server]

our new node name is `dgx` we can confirm it is joined correctly with

```
 kubectl get nodes

```

To disable operands from getting deployed on a GPU worker node, label the node with nvidia.com/gpu.deploy.operands=false.

```
kubectl label nodes dgx  nvidia.com/gpu.deploy.operands=false
```

```
kubectl describe node dgx
kubectl get nodes
```

Then follow the instruction in the [API deployment][TAO-API-DEPLOY] section of the document. 


checks:

Once you run the helm chart you can check `/mnt` directory. The tao presistant volume will be mounted something like `/mnt/nfs_share/default-tao-toolkit-api-pvc-pvc-*` in the GPU node. 


```
kubectl get pods -n default
NAME                                            READY   STATUS    RESTARTS      AGE
ingress-nginx-controller-c69664497-bqzfl        1/1     Running   1 (70s ago)   15h
tao-toolkit-api-app-pod-6c88476548-65sg7        0/1     Pending   0             10s
tao-toolkit-api-workflow-pod-5d57cbf7c5-tv76w   0/1     Pending   0             10s

```


to check the status of PVC(PersistentVolumeClaims)

```
kubectl get pvc

```

answer 

```
kubectl get pvc
NAME                  STATUS    VOLUME   CAPACITY   ACCESS MODES   STORAGECLASS   AGE
tao-toolkit-api-pvc   Pending                                      local          5s

```
This means the **PVC** not yet fulfilled

then we check if there are any Persistent volumes present in our cluster at all.

```
kubectl get pv
``` 

result
```
kubectl get pv
No resources found
```


```
kubectl describe pvc tao-toolkit-api-pvc
```


I had to create a pv, and bind with the pvc claim manually

once bound I can see

```
kubectl get pv
NAME    CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                         STORAGECLASS        REASON   AGE
my-pv   100Gi      RWX            Retain           Bound    default/tao-toolkit-api-pvc   local-storage-dgx            82m

```

status check 


```
kubectl describe pv my-pv
Name:              my-pv
Labels:            <none>
Annotations:       pv.kubernetes.io/bound-by-controller: yes
Finalizers:        [kubernetes.io/pv-protection]
StorageClass:      local-storage-dgx
Status:            Bound
Claim:             default/tao-toolkit-api-pvc
Reclaim Policy:    Retain
Access Modes:      RWX
VolumeMode:        Filesystem
Capacity:          100Gi
Node Affinity:     
  Required Terms:  
    Term 0:        kubernetes.io/hostname in [dgx]
Message:           
Source:
    Type:          HostPath (bare host directory volume)
    Path:          /mnt/k8-local-storage-dgx
    HostPathType:  
Events:            <none>

```


check 1 

```
kubectl get pods -n default
NAME                                            READY   STATUS    RESTARTS       AGE
ingress-nginx-controller-c69664497-bqzfl        1/1     Running   1 (5h2m ago)   20h
tao-toolkit-api-app-pod-6c88476548-2fknz        1/1     Running   0              3m31s
tao-toolkit-api-workflow-pod-5d57cbf7c5-l7dtm   1/1     Running   0              3m31s

```


check 2 


```
kubectl describe pods tao-toolkit-api -n default
Name:             tao-toolkit-api-app-pod-6c88476548-2fknz
Namespace:        default
Priority:         0
Service Account:  default
Node:             dgx/172.16.3.2
Start Time:       Tue, 21 Mar 2023 16:37:41 +0000
Labels:           name=tao-toolkit-api-app-pod
                  pod-template-hash=6c88476548
Annotations:      cni.projectcalico.org/containerID: bf379fa8e6c16ba9aad764cb98b8e949d52a8611cc13e745b71c8ffbf75fc849
                  cni.projectcalico.org/podIP: 192.168.251.146/32
                  cni.projectcalico.org/podIPs: 192.168.251.146/32
Status:           Running
IP:               192.168.251.146
IPs:
  IP:           192.168.251.146
Controlled By:  ReplicaSet/tao-toolkit-api-app-pod-6c88476548
Containers:
  tao-toolkit-api-app:
    Container ID:   containerd://5a8e0ed5702e797bbbc3c86785f19cdaf6e55051a16613c105a01735b7f04205
    Image:          nvcr.io/nvidia/tao/tao-toolkit:4.0.0-api
    Image ID:       nvcr.io/nvidia/tao/tao-toolkit@sha256:db5890fbe2c720ac679a5c1167c495159e4e2bd05cf022f5fbe58bd5c8ad0d8a
    Port:           8000/TCP
    Host Port:      0/TCP
    State:          Running
      Started:      Tue, 21 Mar 2023 16:37:44 +0000
    Ready:          True
    Restart Count:  0
    Liveness:       http-get http://:8000/api/v1/health/liveness delay=3s timeout=3s period=10s #success=1 #failure=3
    Readiness:      http-get http://:8000/api/v1/health/readiness delay=3s timeout=3s period=10s #success=1 #failure=3
    Environment:
      NAMESPACE:        default
      CLAIMNAME:        tao-toolkit-api-pvc
      IMAGEPULLSECRET:  imagepullsecret
      AUTH_CLIENT_ID:   bnSePYullXlG-504nOZeNAXemGF6DhoCdYR8ysm088w
      NUM_GPUS:         1
      BACKEND:          local-k8s
      IMAGE_TF:         nvcr.io/nvidia/tao/tao-toolkit:4.0.0-tf1.15.5
      IMAGE_PYT:        nvcr.io/nvidia/tao/tao-toolkit:4.0.0-pyt
      IMAGE_DNV2:       nvcr.io/nvidia/tao/tao-toolkit:4.0.0-tf1.15.5
      IMAGE_DEFAULT:    nvcr.io/nvidia/tao/tao-toolkit:4.0.0-tf1.15.5
      IMAGE_API:        nvcr.io/nvidia/tao/tao-toolkit:4.0.0-api
    Mounts:
      /shared from shared-data (rw)
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-b2t7v (ro)
Conditions:
  Type              Status
  Initialized       True 
  Ready             True 
  ContainersReady   True 
  PodScheduled      True 
Volumes:
  shared-data:
    Type:       PersistentVolumeClaim (a reference to a PersistentVolumeClaim in the same namespace)
    ClaimName:  tao-toolkit-api-pvc
    ReadOnly:   false
  kube-api-access-b2t7v:
    Type:                    Projected (a volume that contains injected data from multiple sources)
    TokenExpirationSeconds:  3607
    ConfigMapName:           kube-root-ca.crt
    ConfigMapOptional:       <nil>
    DownwardAPI:             true
QoS Class:                   BestEffort
Node-Selectors:              <none>
Tolerations:                 node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                             node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
Events:
  Type     Reason            Age                  From               Message
  ----     ------            ----                 ----               -------
  Warning  FailedScheduling  63s (x2 over 3m58s)  default-scheduler  0/2 nodes are available: pod has unbound immediate PersistentVolumeClaims. preemption: 0/2 nodes are available: 2 No preemption victims found for incoming pod..
  Normal   Scheduled         60s                  default-scheduler  Successfully assigned default/tao-toolkit-api-app-pod-6c88476548-2fknz to dgx
  Normal   Pulling           59s                  kubelet            Pulling image "nvcr.io/nvidia/tao/tao-toolkit:4.0.0-api"
  Normal   Pulled            57s                  kubelet            Successfully pulled image "nvcr.io/nvidia/tao/tao-toolkit:4.0.0-api" in 1.395595053s (2.628318726s including waiting)
  Normal   Created           57s                  kubelet            Created container tao-toolkit-api-app
  Normal   Started           57s                  kubelet            Started container tao-toolkit-api-app

Name:             tao-toolkit-api-workflow-pod-5d57cbf7c5-l7dtm
Namespace:        default
Priority:         0
Service Account:  default
Node:             dgx/172.16.3.2
Start Time:       Tue, 21 Mar 2023 16:37:41 +0000
Labels:           name=tao-toolkit-api-workflow-pod
                  pod-template-hash=5d57cbf7c5
Annotations:      cni.projectcalico.org/containerID: ff93c8258d6844a9bac87536313a6be1387e3d222862359baf6d3acd6ce9c88e
                  cni.projectcalico.org/podIP: 192.168.251.145/32
                  cni.projectcalico.org/podIPs: 192.168.251.145/32
Status:           Running
IP:               192.168.251.145
IPs:
  IP:           192.168.251.145
Controlled By:  ReplicaSet/tao-toolkit-api-workflow-pod-5d57cbf7c5
Containers:
  tao-toolkit-api-workflow:
    Container ID:  containerd://4ed04613f857314d649a7eec9661ab23007e9d93e9fc8a4484759e7fa59eec91
    Image:         nvcr.io/nvidia/tao/tao-toolkit:4.0.0-api
    Image ID:      nvcr.io/nvidia/tao/tao-toolkit@sha256:db5890fbe2c720ac679a5c1167c495159e4e2bd05cf022f5fbe58bd5c8ad0d8a
    Port:          <none>
    Host Port:     <none>
    Command:
      /bin/bash
      workflow_start.sh
    State:          Running
      Started:      Tue, 21 Mar 2023 16:37:43 +0000
    Ready:          True
    Restart Count:  0
    Environment:
      NAMESPACE:               default
      CLAIMNAME:               tao-toolkit-api-pvc
      IMAGEPULLSECRET:         imagepullsecret
      NUM_GPUS:                1
      TELEMETRY_OPT_OUT:       no
      BACKEND:                 local-k8s
      IMAGE_TF:                nvcr.io/nvidia/tao/tao-toolkit:4.0.0-tf1.15.5
      IMAGE_PYT:               nvcr.io/nvidia/tao/tao-toolkit:4.0.0-pyt
      IMAGE_DNV2:              nvcr.io/nvidia/tao/tao-toolkit:4.0.0-tf1.15.5
      IMAGE_DEFAULT:           nvcr.io/nvidia/tao/tao-toolkit:4.0.0-tf1.15.5
      IMAGE_API:               nvcr.io/nvidia/tao/tao-toolkit:4.0.0-api
      WANDB_API_KEY:           
      CLEARML_WEB_HOST:        https://app.clear.ml
      CLEARML_API_HOST:        https://api.clear.ml
      CLEARML_FILES_HOST:      https://files.clear.ml
      CLEARML_API_ACCESS_KEY:  
      CLEARML_API_SECRET_KEY:  
    Mounts:
      /shared from shared-data (rw)
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-w6d8r (ro)
Conditions:
  Type              Status
  Initialized       True 
  Ready             True 
  ContainersReady   True 
  PodScheduled      True 
Volumes:
  shared-data:
    Type:       PersistentVolumeClaim (a reference to a PersistentVolumeClaim in the same namespace)
    ClaimName:  tao-toolkit-api-pvc
    ReadOnly:   false
  kube-api-access-w6d8r:
    Type:                    Projected (a volume that contains injected data from multiple sources)
    TokenExpirationSeconds:  3607
    ConfigMapName:           kube-root-ca.crt
    ConfigMapOptional:       <nil>
    DownwardAPI:             true
QoS Class:                   BestEffort
Node-Selectors:              <none>
Tolerations:                 node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                             node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
Events:
  Type     Reason            Age                  From               Message
  ----     ------            ----                 ----               -------
  Warning  FailedScheduling  63s (x2 over 3m58s)  default-scheduler  0/2 nodes are available: pod has unbound immediate PersistentVolumeClaims. preemption: 0/2 nodes are available: 2 No preemption victims found for incoming pod..
  Normal   Scheduled         60s                  default-scheduler  Successfully assigned default/tao-toolkit-api-workflow-pod-5d57cbf7c5-l7dtm to dgx
  Normal   Pulling           59s                  kubelet            Pulling image "nvcr.io/nvidia/tao/tao-toolkit:4.0.0-api"
  Normal   Pulled            58s                  kubelet            Successfully pulled image "nvcr.io/nvidia/tao/tao-toolkit:4.0.0-api" in 1.278229741s (1.278312255s including waiting)
  Normal   Created           58s                  kubelet            Created container tao-toolkit-api-workflow
  Normal   Started           58s                  kubelet            Started container tao-toolkit-api-workflow

```
files needed:

1. custom-volume-definition.yaml

```
apiVersion: v1
kind: PersistentVolume
metadata:
  name: my-pv
spec:
  storageClassName: custom-local-storage
  capacity:
    storage: 100Gi
  accessModes:
    - ReadWriteMany
  hostPath:
    path: /mnt/k8-local-storage-dgx
  nodeAffinity:
    required:
      nodeSelectorTerms:
        - matchExpressions:
            - key: kubernetes.io/hostname
              operator: In
              values:
                - dgx
  persistentVolumeReclaimPolicy: Retain
  volumeMode: Filesystem
``` 

2. local-storage-declaration.yaml

```
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: custom-local-storage
provisioner: kubernetes.io/no-provisioner
volumeBindingMode: Immediate
allowVolumeExpansion: true
parameters:
  type: local-storage

```

2. values.yaml(modified)

```
# TAO Toolkit API container info
image: nvcr.io/nvidia/tao/tao-toolkit:4.0.0-api
imagePullSecret: imagepullsecret
imagePullPolicy: Always 

# Optional HTTPS settings for ingress controller
#host: mydomain.com
#tlsSecret: tls-secret
#corsOrigin: https://mydomain.com

# Shared storage info
storageClassName: custom-local-storage
storageAccessMode: ReadWriteMany
storageSize: 100Gi
ephemeral-storage: 8Gi
limits.ephemeral-storage: 8Gi
requests.ephemeral-storage: 4Gi

# Optional NVIDIA Starfleet authentication
#authClientId: bnSePYullXlG-504nOZeNAXemGF6DhoCdYR8ysm088w

# Starting TAO Toolkit jobs info
backend: local-k8s
numGpus: 1
imageTf: nvcr.io/nvidia/tao/tao-toolkit:4.0.0-tf1.15.5
imagePyt: nvcr.io/nvidia/tao/tao-toolkit:4.0.0-pyt
imageDnv2: nvcr.io/nvidia/tao/tao-toolkit:4.0.0-tf1.15.5
imageDefault: nvcr.io/nvidia/tao/tao-toolkit:4.0.0-tf1.15.5
```


At this point [user authentication][TAO-API-USER_AUTHENTICATION] should work and in the mount point you will see newly downloaded artifacts check with (`sudo du /mnt/k8-local-storage-dgx/ -h -d 3
`)


P.S.


helm install with overrides (note: the README file in the tao-toolkit-api folder is very insightful)
```
helm install tao-toolkit-api tao-toolkit-api/ --namespace default --values tao-toolkit-api/values.yaml
```


get pvc uid 
```
kubectl get pvc my-pvc -o jsonpath='{.metadata.uid}'

```

manually bind volume
```
kubectl patch pv my-pv -p '{"spec":{"claimRef":{"name":"tao-toolkit-api-pvc","namespace":"default","uid":"c231660c-5e36-4417-b304-f0b4a1198a7c"}}}'
```






<br />

<span style="background-color:LightYellow"> Check the [**next topic**](../pytorch_walkthrough#Starting-Development-with-PyTorch)  </span>


---
*Click [here][ERRORS-SUGGESTIONS] to report Errors, make Suggestions or Comments!*

[NVIDIA-INSTALL-K8]: https://docs.nvidia.com/datacenter/cloud-native/kubernetes/install-k8s.html#option-2-installing-kubernetes-using-kubeadm
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
[INSTALL-HELM]: https://helm.sh/docs/intro/install/#from-script
[INSTALL-NVIDIA-GPU-OPERATOR]: https://docs.nvidia.com/datacenter/cloud-native/gpu-operator/getting-started.html#install-nvidia-gpu-operator
[INSTALL-NVIDIA-GPU-OPERATOR-NO_DRIVERS]: https://docs.nvidia.com/datacenter/cloud-native/gpu-operator/getting-started.html#bare-metal-passthrough-with-pre-installed-nvidia-container-toolkit-but-no-drivers

[TAO-API-DEPLOY]: https://docs.nvidia.com/tao/tao-toolkit/text/tao_toolkit_api/api_deployment.html
[TAO-API-USER_AUTHENTICATION]: https://docs.nvidia.com/tao/tao-toolkit/text/tao_toolkit_api/api_rest_api.html#user-authentication

[ERRORS-SUGGESTIONS]: https://github.com/ganindu7/deepnotes/issues
[CRI-DOCKERD-RELEASE-JAN-2023]: https://github.com/Mirantis/cri-dockerd/releases/download/v0.3.1/cri-dockerd-0.3.1.amd64.tgz
[K8-SANDBOX]: https://labs.play-with-k8s.com/
[K8-CLASSROOM]: https://training.play-with-kubernetes.com/kubernetes-workshop/
[SSH-KEY-MAKING]: https://docs.github.com/en/authentication/connecting-to-github-with-ssh/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent
[ADD-SSHKEY-TO-AGENT]: https://docs.github.com/en/authentication/connecting-to-github-with-ssh/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent#adding-your-ssh-key-to-the-ssh-agent

<!-- Latex in markdown -->
<script src="https://cdn.mathjax.org/mathjax/latest/MathJax.js?config=TeX-AMS-MML_HTMLorMML" type="text/javascript"></script>
<!-- $$ \nabla_\boldsymbol{x} J(\boldsymbol{x}) $$ -->
