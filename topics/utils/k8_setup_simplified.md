---
layout: default
title: simplified k8 install instructions 
nav_order: 2 
# permalink: /topics/utils/target_execute
parent: Utilities
---

## Installing Kubernetes on DGX using Kubeadm (Updated for TAO 5.0.0)
<span style="background-color:LightGreen">
Created : 18/05/2023 <br />
Status: Draft
</span>


### Writeup


These are simplified instructions to install k8 and services 

my installed k8 version 

```
echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.24/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list
apt-cache policy kubectl 
sudo apt-get install -y kubelet=1.24.2-1.1 kubeadm=1.24.2-1.1  kubectl=1.24.2-1.1
sudo apt-mark hold kubelet kubeadm kubectl

```


install 1.23.5-00

1. add key (this method may be depricated)

```
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
```
2. update souces list 

```
cat << EOF | sudo tee /etc/apt/sources.list.d/kubernetes.list
deb https://apt.kubernetes.io/ kubernetes-xenial main
EOF
```

3. update packakge list and check apt cache to make sure we have the version we are looking for 

```
sudo apt update
apt-cache policy kubeadm
```

when you scroll down you'll see 

```
1.23.5-00 500
        500 https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages

```


3. install 

```
sudo apt-get install -y kubelet=1.24.14-00 kubeadm=1.24.14-00 kubectl=1.24.14-00 --allow-downgrades --allow-change-held-packages

sudo apt-get install -y kubelet=1.23.5-00 kubeadm=1.23.5-00 kubectl=1.23.5-00 --allow-downgrades --allow-change-held-packages
```

### If you had previous installations 

official guidance can be found here [here](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/#tear-down)

```
drain all nodes 
```

reset kubeadm


normal setup


```
sudo kubeadm reset 
```

specifiying the CRI socket 

```
sudo kubeadm reset --cri-socket=unix:///var/run/cri-dockerd.sock
```


rest changes to networking

```
sudo rm -rf /etc/cni/net.d
rm -rf $HOME/.kube

```

### Clear IP tables 


basic command

```
sudo iptables -F 
sudo iptables -t nat -F  
sudo iptables -t mangle -F 
sudo iptables -X

```

if it doesn't work


#### Set default policies
```
sudo iptables -P INPUT ACCEPT
sudo iptables -P FORWARD ACCEPT
sudo iptables -P OUTPUT ACCEPT
```

#### Flush all rules
```
sudo iptables -t filter -F
sudo iptables -t nat -F
sudo iptables -t mangle -F
sudo iptables -t raw -F
sudo iptables -t security -F
```

#### Delete all non-default chains
```
sudo iptables -t filter -X
sudo iptables -t nat -X
sudo iptables -t mangle -X
sudo iptables -t raw -X
sudo iptables -t security -X

```



### Fresh install 

run the command in the master node (control plane)

without specifying the socket

```
sudo kubeadm init --pod-network-cidr=192.168.0.0/16 

```

with specifying the socket 

```
sudo kubeadm init --pod-network-cidr=192.168.0.0/16 --cri-socket=unix:///var/run/cri-dockerd.sock 

```

for a specific k8 version try:


```

sudo kubeadm init --pod-network-cidr=192.168.0.0/16 --kubernetes-version=v1.23.5


```

Note: cri-dockered is not compatible with version 1.23 

setup 

```
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config

```

setup network stuff 

[install calico](https://docs.tigera.io/calico/latest/getting-started/kubernetes/helm)


install calico and the install calicoctl 


note: 

my values.yaml

```
imagePullSecrets: {}

installation:
  enabled: true
  kubernetesProvider:
  cni:
    type: Calico

apiServer:
  enabled: true

certs:
  node:
    key:
    cert:
    commonName:
  typha:
    key:
    cert:
    commonName:
    caBundle:

resources: {}

tolerations:
- effect: NoExecute
  operator: Exists
- effect: NoSchedule
  operator: Exists

nodeSelector:
  kubernetes.io/os: linux

podAnnotations: {}

podLabels: {}

tigeraOperator:
  image: tigera/operator
  version: v1.29.3
  registry: quay.io
calicoctl:
  image: docker.io/calico/ctl
  tag: v3.25.1

calicoNetwork:
  bgp: Enabled
  ipPools:
  - cidr: 192.168.0.0/16
    encapsulation: VXLAN
    natOutgoing: Enabled
    nodeSelector: all()

```


confirm all pods are working 

```
watch kubectl get pods -n calico-system
```

check the IP pool 

```
kubectl calico ipam show
```

or if the cluster and calicoctl versions do not match

```
kubectl-calico ipam show --allow-version-mismatch
```

you will get something like 

```
+----------+----------------+-----------+------------+--------------+
| GROUPING |      CIDR      | IPS TOTAL | IPS IN USE |   IPS FREE   |
+----------+----------------+-----------+------------+--------------+
| IP Pool  | 192.168.0.0/16 |     65536 | 7 (0%)     | 65529 (100%) |
+----------+----------------+-----------+------------+--------------+
```

get calicotl version pods with `kubectl-calico version` or `calicoctl version`


The aim is to make sure the cluster and the client have the same version I get something

```
kubectl-calico version
Client Version:    v3.25.1
Git commit:        82dadbce1
Cluster Version:   v3.25.1
Cluster Type:      typha,kdd,k8s,operator,bgp,kubeadm
```

### in the worker node (do this before resetting the master)

if it was previously used ssh into that and run 

```
sudo kubeadm reset
```

to reset the node, then clean up networking configs 

```
sudo rm -rf /etc/cni/net.d
rm -rf ~/.kube
```

if that fails 

1. stop the services 
```
sudo systemctl stop kubelet
sudo systemctl stop <container-runtime-service> (containerd or dockerd)
```

delete settings 
```
sudo rm -rf /etc/kubernetes
sudo rm -rf /var/lib/kubelet
sudo rm -rf /var/lib/etcd
sudo rm -rf /var/lib/cni
sudo rm -rf /etc/cni/net.d
sudo rm -rf /var/run/kubernetes
```

clear iptables and firewall

```
sudo iptables -P INPUT ACCEPT
sudo iptables -P FORWARD ACCEPT
sudo iptables -P OUTPUT ACCEPT
sudo iptables -t filter -F
sudo iptables -t nat -F
sudo iptables -t mangle -F
sudo iptables -t raw -F
sudo iptables -t security -F
sudo iptables -t filter -X
sudo iptables -t nat -X
sudo iptables -t mangle -X
sudo iptables -t raw -X
sudo iptables -t security -X
```

list the containers (view running containers)
```
sudo crictl --runtime-endpoint unix:///run/containerd/containerd.sock ps -a
```

list only the container IDs (list running contianer IDS)
```
sudo crictl --runtime-endpoint unix:///run/containerd/containerd.sock ps -aq
```
delete all containers (one liner, this is not tested enough) 
```
sudo crictl --runtime-endpoint unix:///run/containerd/containerd.sock ps -aq | xargs -r -I {} sudo crictl --runtime-endpoint unix:///run/containerd/containerd.sock rm {}
```

if everything above fails try 

clear downloaded containers (this step is not really needed)
```
sudo rm -rf /var/lib/docker
sudo rm -rf /var/lib/containerd
```

then 

```
sudo reboot and try kubeadm reset again
```


Onve the node is properly resetted get a join token from the master node and apply to the user node  

```
kubeadm token create --print-join-command
```

to be able to use `kubectl` from the worker node copy the `$HOME/.kube/config` to the worker (optional)


modified  `/etc/containerd/config.toml` (to be able to use the gpu operator)

cexample can alo be found at [thenvidia repo](https://raw.githubusercontent.com/NVIDIA/cloud-native-stack/master/playbooks/files/config.toml)


```
disabled_plugins = []
imports = []
oom_score = 0
plugin_dir = ""
required_plugins = []
root = "/var/lib/containerd"
state = "/run/containerd"
temp = ""
version = 2

[cgroup]
  path = ""

[debug]
  address = ""
  format = ""
  gid = 0
  level = ""
  uid = 0

[grpc]
  address = "/run/containerd/containerd.sock"
  gid = 0
  max_recv_message_size = 16777216
  max_send_message_size = 16777216
  tcp_address = ""
  tcp_tls_ca = ""
  tcp_tls_cert = ""
  tcp_tls_key = ""
  uid = 0

[metrics]
  address = ""
  grpc_histogram = false

[plugins]

  [plugins."io.containerd.gc.v1.scheduler"]
    deletion_threshold = 0
    mutation_threshold = 100
    pause_threshold = 0.02
    schedule_delay = "0s"
    startup_delay = "100ms"

  [plugins."io.containerd.grpc.v1.cri"]
    device_ownership_from_security_context = false
    disable_apparmor = false
    disable_cgroup = false
    disable_hugetlb_controller = true
    disable_proc_mount = false
    disable_tcp_service = true
    enable_selinux = false
    enable_tls_streaming = false
    enable_unprivileged_icmp = false
    enable_unprivileged_ports = false
    ignore_image_defined_volumes = false
    max_concurrent_downloads = 3
    max_container_log_line_size = 16384
    netns_mounts_under_state_dir = false
    restrict_oom_score_adj = false
    sandbox_image = "k8s.gcr.io/pause:3.6"
    selinux_category_range = 1024
    stats_collect_period = 10
    stream_idle_timeout = "4h0m0s"
    stream_server_address = "127.0.0.1"
    stream_server_port = "0"
    systemd_cgroup = false
    tolerate_missing_hugetlb_controller = true
    unset_seccomp_profile = ""

    [plugins."io.containerd.grpc.v1.cri".cni]
      bin_dir = "/opt/cni/bin"
      conf_dir = "/etc/cni/net.d"
      conf_template = ""
      ip_pref = ""
      max_conf_num = 1

    [plugins."io.containerd.grpc.v1.cri".containerd]
      default_runtime_name = "nvidia"
      disable_snapshot_annotations = true
      discard_unpacked_layers = false
      ignore_rdt_not_enabled_errors = false
      no_pivot = false
      snapshotter = "overlayfs"

      [plugins."io.containerd.grpc.v1.cri".containerd.default_runtime]
        base_runtime_spec = ""
        cni_conf_dir = ""
        cni_max_conf_num = 0
        container_annotations = []
        pod_annotations = []
        privileged_without_host_devices = false
        runtime_engine = ""
        runtime_path = ""
        runtime_root = ""
        runtime_type = ""

        [plugins."io.containerd.grpc.v1.cri".containerd.default_runtime.options]

      [plugins."io.containerd.grpc.v1.cri".containerd.runtimes]

        [plugins."io.containerd.grpc.v1.cri".containerd.runtimes.nvidia]
          base_runtime_spec = ""
          cni_conf_dir = ""
          cni_max_conf_num = 0
          container_annotations = []
          pod_annotations = []
          privileged_without_host_devices = false
          runtime_engine = ""
          runtime_path = ""
          runtime_root = ""
          runtime_type = "io.containerd.runc.v2"

          [plugins."io.containerd.grpc.v1.cri".containerd.runtimes.nvidia.options]
            BinaryName = "/usr/local/nvidia/toolkit/nvidia-container-runtime"
            CriuImagePath = ""
            CriuPath = ""
            CriuWorkPath = ""
            IoGid = 0
            IoUid = 0
            NoNewKeyring = false
            NoPivotRoot = false
            Root = ""
            ShimCgroup = ""
            SystemdCgroup = true

        [plugins."io.containerd.grpc.v1.cri".containerd.runtimes.nvidia-cdi]
          base_runtime_spec = ""
          cni_conf_dir = ""
          cni_max_conf_num = 0
          container_annotations = []
          pod_annotations = []
          privileged_without_host_devices = false
          runtime_engine = ""
          runtime_path = ""
          runtime_root = ""
          runtime_type = "io.containerd.runc.v2"

          [plugins."io.containerd.grpc.v1.cri".containerd.runtimes.nvidia-cdi.options]
            BinaryName = "/usr/local/nvidia/toolkit/nvidia-container-runtime.cdi"
            CriuImagePath = ""
            CriuPath = ""
            CriuWorkPath = ""
            IoGid = 0
            IoUid = 0
            NoNewKeyring = false
            NoPivotRoot = false
            Root = ""
            ShimCgroup = ""
            SystemdCgroup = true

        [plugins."io.containerd.grpc.v1.cri".containerd.runtimes.nvidia-experimental]
          base_runtime_spec = ""
          cni_conf_dir = ""
          cni_max_conf_num = 0
          container_annotations = []
          pod_annotations = []
          privileged_without_host_devices = false
          runtime_engine = ""
          runtime_path = ""
          runtime_root = ""
          runtime_type = "io.containerd.runc.v2"

          [plugins."io.containerd.grpc.v1.cri".containerd.runtimes.nvidia-experimental.options]
            BinaryName = "/usr/local/nvidia/toolkit/nvidia-container-runtime.experimental"
            CriuImagePath = ""
            CriuPath = ""
            CriuWorkPath = ""
            IoGid = 0
            IoUid = 0
            NoNewKeyring = false
            NoPivotRoot = false
            Root = ""
            ShimCgroup = ""
            SystemdCgroup = true

        [plugins."io.containerd.grpc.v1.cri".containerd.runtimes.nvidia-legacy]
          base_runtime_spec = ""
          cni_conf_dir = ""
          cni_max_conf_num = 0
          container_annotations = []
          pod_annotations = []
          privileged_without_host_devices = false
          runtime_engine = ""
          runtime_path = ""
          runtime_root = ""
          runtime_type = "io.containerd.runc.v2"

          [plugins."io.containerd.grpc.v1.cri".containerd.runtimes.nvidia-legacy.options]
            BinaryName = "/usr/local/nvidia/toolkit/nvidia-container-runtime.legacy"
            CriuImagePath = ""
            CriuPath = ""
            CriuWorkPath = ""
            IoGid = 0
            IoUid = 0
            NoNewKeyring = false
            NoPivotRoot = false
            Root = ""
            ShimCgroup = ""
            SystemdCgroup = true

        [plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc]
          base_runtime_spec = ""
          cni_conf_dir = ""
          cni_max_conf_num = 0
          container_annotations = []
          pod_annotations = []
          privileged_without_host_devices = false
          runtime_engine = ""
          runtime_path = ""
          runtime_root = ""
          runtime_type = "io.containerd.runc.v2"

          [plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc.options]
            BinaryName = ""
            CriuImagePath = ""
            CriuPath = ""
            CriuWorkPath = ""
            IoGid = 0
            IoUid = 0
            NoNewKeyring = false
            NoPivotRoot = false
            Root = ""
            ShimCgroup = ""
            SystemdCgroup = true

      [plugins."io.containerd.grpc.v1.cri".containerd.untrusted_workload_runtime]
        base_runtime_spec = ""
        cni_conf_dir = ""
        cni_max_conf_num = 0
        container_annotations = []
        pod_annotations = []
        privileged_without_host_devices = false
        runtime_engine = ""
        runtime_path = ""
        runtime_root = ""
        runtime_type = ""

        [plugins."io.containerd.grpc.v1.cri".containerd.untrusted_workload_runtime.options]

    [plugins."io.containerd.grpc.v1.cri".image_decryption]
      key_model = "node"

    [plugins."io.containerd.grpc.v1.cri".registry]
      config_path = ""

      [plugins."io.containerd.grpc.v1.cri".registry.auths]

      [plugins."io.containerd.grpc.v1.cri".registry.configs]

      [plugins."io.containerd.grpc.v1.cri".registry.headers]

      [plugins."io.containerd.grpc.v1.cri".registry.mirrors]

    [plugins."io.containerd.grpc.v1.cri".x509_key_pair_streaming]
      tls_cert_file = ""
      tls_key_file = ""

  [plugins."io.containerd.internal.v1.opt"]
    path = "/opt/containerd"

  [plugins."io.containerd.internal.v1.restart"]
    interval = "10s"

  [plugins."io.containerd.internal.v1.tracing"]
    sampling_ratio = 1.0
    service_name = "containerd"

  [plugins."io.containerd.metadata.v1.bolt"]
    content_sharing_policy = "shared"

  [plugins."io.containerd.monitor.v1.cgroups"]
    no_prometheus = false

  [plugins."io.containerd.runtime.v1.linux"]
    no_shim = false
    runtime = "runc"
    runtime_root = ""
    shim = "containerd-shim"
    shim_debug = false

  [plugins."io.containerd.runtime.v2.task"]
    platforms = ["linux/amd64"]
    sched_core = false

  [plugins."io.containerd.service.v1.diff-service"]
    default = ["walking"]

  [plugins."io.containerd.service.v1.tasks-service"]
    rdt_config_file = ""

  [plugins."io.containerd.snapshotter.v1.aufs"]
    root_path = ""

  [plugins."io.containerd.snapshotter.v1.btrfs"]
    root_path = ""

  [plugins."io.containerd.snapshotter.v1.devmapper"]
    async_remove = false
    base_image_size = ""
    discard_blocks = false
    fs_options = ""
    fs_type = ""
    pool_name = ""
    root_path = ""

  [plugins."io.containerd.snapshotter.v1.native"]
    root_path = ""

  [plugins."io.containerd.snapshotter.v1.overlayfs"]
    root_path = ""
    upperdir_label = false

  [plugins."io.containerd.snapshotter.v1.zfs"]
    root_path = ""

  [plugins."io.containerd.tracing.processor.v1.otlp"]
    endpoint = ""
    insecure = false
    protocol = ""

[proxy_plugins]

[stream_processors]

  [stream_processors."io.containerd.ocicrypt.decoder.v1.tar"]
    accepts = ["application/vnd.oci.image.layer.v1.tar+encrypted"]
    args = ["--decryption-keys-path", "/etc/containerd/ocicrypt/keys"]
    env = ["OCICRYPT_KEYPROVIDER_CONFIG=/etc/containerd/ocicrypt/ocicrypt_keyprovider.conf"]
    path = "ctd-decoder"
    returns = "application/vnd.oci.image.layer.v1.tar"

  [stream_processors."io.containerd.ocicrypt.decoder.v1.tar.gzip"]
    accepts = ["application/vnd.oci.image.layer.v1.tar+gzip+encrypted"]
    args = ["--decryption-keys-path", "/etc/containerd/ocicrypt/keys"]
    env = ["OCICRYPT_KEYPROVIDER_CONFIG=/etc/containerd/ocicrypt/ocicrypt_keyprovider.conf"]
    path = "ctd-decoder"
    returns = "application/vnd.oci.image.layer.v1.tar+gzip"

[timeouts]
  "io.containerd.timeout.bolt.open" = "0s"
  "io.containerd.timeout.shim.cleanup" = "5s"
  "io.containerd.timeout.shim.load" = "5s"
  "io.containerd.timeout.shim.shutdown" = "3s"
  "io.containerd.timeout.task.state" = "2s"

[ttrpc]
  address = ""
  gid = 0
  uid = 0


```
## Install the gpu operator 

follow [these instructions](https://docs.nvidia.com/datacenter/cloud-native/gpu-operator/latest/getting-started.html#bare-metal-passthrough-with-pre-installed-drivers-and-nvidia-container-toolkit) to install gpu-operator. 



make sure to wait unitl the node is in ready state beforehand and wait for gpu-operator pods to install in all the nodes. 

hint: if networking pods throw error e.g. "kubernetes-worker-node-is-notready-due-to-cni-plugin-not-initialized" restart the containerd or dockerd status with e.g. `sudo systemctl restart   containerd`


```
helm install --wait --generate-name \
     -n gpu-operator --create-namespace \
      nvidia/gpu-operator \
      --set driver.enabled=false \
      --set toolkit.enabled=false
```

## Installing the k8 metrics server 

```
helm repo add metrics-server https://kubernetes-sigs.github.io/metrics-server/
helm upgrade --install metrics-server metrics-server/metrics-server --namespace k8-metrics --create-namespace
```

once installed make sure the api-service is running:

```
kubectl get apiservices
```

if it is not: edit the deployment and add the following 

```
 - --kubelet-insecure-tls
```

the deployment will then look like 

```

  containers:
      - args:
        - --secure-port=10250
        - --cert-dir=/tmp
        - --kubelet-preferred-address-types=InternalIP,ExternalIP,Hostname
        - --kubelet-use-node-status-port
        - --metric-resolution=15s
        - --kubelet-insecure-tls

```
      


## installing the k8 dashboard 

```
helm install kubernetes-dashboard kubernetes-dashboard/kubernetes-dashboard --namespace kubernetes-dashboard -f valus.yaml --create-namespace
```

this `values.yaml` will create a NodePort service that can be accessed via the port, token TTL is for the lifetime of the auth token. When the existing token expires 
you can use `kubectl -n kubernetes-dashboard create token admin-user` to create a new token. 

```
service:
  type: NodePort
  nodePort: 30001 # You can change the port number according to your needs
  tokenTTL: 28800

ingress:
  enabled: false
```

--------------------

Notes:

I wanted to modify the yaml templates that take overriding values from the `values.yaml` so i made a backup file (e.g. ingress.yaml to ingress.yaml.backup) But I noticed that the created ingresses had the incorrect class name. then i renamed the backup to `ingress-yaml.backup` then it all worked. this means it only looks for the filename.yaml tag 


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

