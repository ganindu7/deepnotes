---
layout: default
title: Dgx as a GPU node with gpu operator 
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

### If you had previous installations 

```
drain all nodes 
```

reset kubeadm

```
sudo kubeadm reset --cri-socket=unix:///var/run/cri-dockerd.sock
```

rest changes to networking

```
rm -rf /etc/cni/net.d
rm -rf $HOME/.kube

sudo iptables -P INPUT ACCEPT
sudo iptables -P FORWARD ACCEPT
sudo iptables -P OUTPUT ACCEPT
sudo iptables -t nat -F
sudo iptables -t mangle -F
sudo iptables -F
sudo iptables -X

```

### Fresh install 

```
sudo kubeadm init --pod-network-cidr=192.168.0.0/16 --cri-socket=unix:///var/run/cri-dockerd.sock

```
setup 

```
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config

```

setup network stuff 

[install calico](https://docs.tigera.io/calico/latest/getting-started/kubernetes/helm)

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

### in the worker node 

if it was previously used ssh into that and run 

```
sudo kubeadm reset
```

to rest the node, then clean up networking configs 

```
sudo rm -rf /etc/cni/net.d
```

once it is all clean, reset kubeadm 

```
sudo kubeadm reset
```

if that fails 

```
sudo systemctl stop kubelet
sudo systemctl stop <container-runtime-service> (containerd or dockerd)

sudo rm -rf /etc/kubernetes
sudo rm -rf /var/lib/kubelet
sudo rm -rf /var/lib/etcd
sudo rm -rf /var/lib/cni
sudo rm -rf /etc/cni/net.d
sudo rm -rf /var/run/kubernetes
sudo rm -rf /var/lib/docker
sudo rm -rf /var/lib/containerd

then 

sudo reboot and try kubeadm reset again

```

modified  `/etc/containerd/config.toml`


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
    sandbox_image = "registry.k8s.io/pause:3.6"
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

      [plugins."io.containerd.grpc.v1.cri".containerd.runtimes.nvidia]
        privileged_without_host_devices = false
        runtime_engine = ""
        runtime_root = ""
    runtime_type = "io.containerd.runc.v2"
      [plugins."io.containerd.grpc.v1.cri".containerd.runtimes.nvidia.options]
    BinaryName = "/usr/bin/nvidia-container-runtime"
    SystemdCgroup = true
      

        [plugins."io.containerd.grpc.v1.cri".containerd.default_runtime.options]

      [plugins."io.containerd.grpc.v1.cri".containerd.runtimes]

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



then add the dgx node with the join command 

install gpu operator with [these instructions](https://docs.nvidia.com/datacenter/cloud-native/gpu-operator/getting-started.html#bare-metal-passthrough-with-pre-installed-drivers-and-nvidia-container-toolkit)


Note: I've used the config file as my `/etc/containerd/config.toml`


when working gpu node 

```
kubectl describe node dgx
Name:               dgx
Roles:              <none>
Labels:             beta.kubernetes.io/arch=amd64
                    beta.kubernetes.io/os=linux
                    feature.node.kubernetes.io/cpu-cpuid.ADX=true
                    feature.node.kubernetes.io/cpu-cpuid.AESNI=true
                    feature.node.kubernetes.io/cpu-cpuid.AVX=true
                    feature.node.kubernetes.io/cpu-cpuid.AVX2=true
                    feature.node.kubernetes.io/cpu-cpuid.CLZERO=true
                    feature.node.kubernetes.io/cpu-cpuid.CPBOOST=true
                    feature.node.kubernetes.io/cpu-cpuid.FMA3=true
                    feature.node.kubernetes.io/cpu-cpuid.IBS=true
                    feature.node.kubernetes.io/cpu-cpuid.IBSBRNTRGT=true
                    feature.node.kubernetes.io/cpu-cpuid.IBSFETCHSAM=true
                    feature.node.kubernetes.io/cpu-cpuid.IBSFFV=true
                    feature.node.kubernetes.io/cpu-cpuid.IBSOPCNT=true
                    feature.node.kubernetes.io/cpu-cpuid.IBSOPCNTEXT=true
                    feature.node.kubernetes.io/cpu-cpuid.IBSOPSAM=true
                    feature.node.kubernetes.io/cpu-cpuid.IBSRDWROPCNT=true
                    feature.node.kubernetes.io/cpu-cpuid.IBSRIPINVALIDCHK=true
                    feature.node.kubernetes.io/cpu-cpuid.INT_WBINVD=true
                    feature.node.kubernetes.io/cpu-cpuid.MCAOVERFLOW=true
                    feature.node.kubernetes.io/cpu-cpuid.MCOMMIT=true
                    feature.node.kubernetes.io/cpu-cpuid.MSRIRC=true
                    feature.node.kubernetes.io/cpu-cpuid.RDPRU=true
                    feature.node.kubernetes.io/cpu-cpuid.SHA=true
                    feature.node.kubernetes.io/cpu-cpuid.SSE4A=true
                    feature.node.kubernetes.io/cpu-cpuid.SUCCOR=true
                    feature.node.kubernetes.io/cpu-cpuid.WBNOINVD=true
                    feature.node.kubernetes.io/cpu-hardware_multithreading=true
                    feature.node.kubernetes.io/cpu-rdt.RDTCMT=true
                    feature.node.kubernetes.io/cpu-rdt.RDTL3CA=true
                    feature.node.kubernetes.io/cpu-rdt.RDTMBM=true
                    feature.node.kubernetes.io/cpu-rdt.RDTMON=true
                    feature.node.kubernetes.io/custom-rdma.available=true
                    feature.node.kubernetes.io/kernel-config.NO_HZ=true
                    feature.node.kubernetes.io/kernel-config.NO_HZ_IDLE=true
                    feature.node.kubernetes.io/kernel-version.full=5.4.0-146-generic
                    feature.node.kubernetes.io/kernel-version.major=5
                    feature.node.kubernetes.io/kernel-version.minor=4
                    feature.node.kubernetes.io/kernel-version.revision=0
                    feature.node.kubernetes.io/network-sriov.capable=true
                    feature.node.kubernetes.io/pci-10de.present=true
                    feature.node.kubernetes.io/pci-10de.sriov.capable=true
                    feature.node.kubernetes.io/pci-1a03.present=true
                    feature.node.kubernetes.io/pci-8086.present=true
                    feature.node.kubernetes.io/pci-8086.sriov.capable=true
                    feature.node.kubernetes.io/storage-nonrotationaldisk=true
                    feature.node.kubernetes.io/system-os_release.ID=ubuntu
                    feature.node.kubernetes.io/system-os_release.VERSION_ID=20.04
                    feature.node.kubernetes.io/system-os_release.VERSION_ID.major=20
                    feature.node.kubernetes.io/system-os_release.VERSION_ID.minor=04
                    kubernetes.io/arch=amd64
                    kubernetes.io/hostname=dgx
                    kubernetes.io/os=linux
                    nvidia.com/cuda.driver.major=525
                    nvidia.com/cuda.driver.minor=85
                    nvidia.com/cuda.driver.rev=12
                    nvidia.com/cuda.runtime.major=12
                    nvidia.com/cuda.runtime.minor=0
                    nvidia.com/gfd.timestamp=1680714586
                    nvidia.com/gpu.compute.major=8
                    nvidia.com/gpu.compute.minor=0
                    nvidia.com/gpu.count=4
                    nvidia.com/gpu.deploy.container-toolkit=true
                    nvidia.com/gpu.deploy.dcgm=true
                    nvidia.com/gpu.deploy.dcgm-exporter=true
                    nvidia.com/gpu.deploy.device-plugin=true
                    nvidia.com/gpu.deploy.driver=true
                    nvidia.com/gpu.deploy.gpu-feature-discovery=true
                    nvidia.com/gpu.deploy.mig-manager=true
                    nvidia.com/gpu.deploy.node-status-exporter=true
                    nvidia.com/gpu.deploy.operator-validator=true
                    nvidia.com/gpu.family=ampere
                    nvidia.com/gpu.machine=DGX-Station-A100-920-23487-2530-0R0
                    nvidia.com/gpu.memory=40960
                    nvidia.com/gpu.present=true
                    nvidia.com/gpu.product=NVIDIA-A100-SXM4-40GB
                    nvidia.com/gpu.replicas=1
                    nvidia.com/mig.capable=true
                    nvidia.com/mig.strategy=single
Annotations:        csi.volume.kubernetes.io/nodeid: {"csi.tigera.io":"dgx"}
                    kubeadm.alpha.kubernetes.io/cri-socket: unix:///var/run/containerd/containerd.sock
                    nfd.node.kubernetes.io/extended-resources: 
                    nfd.node.kubernetes.io/feature-labels:
                      cpu-cpuid.ADX,cpu-cpuid.AESNI,cpu-cpuid.AVX,cpu-cpuid.AVX2,cpu-cpuid.CLZERO,cpu-cpuid.CPBOOST,cpu-cpuid.FMA3,cpu-cpuid.IBS,cpu-cpuid.IBSBR...
                    nfd.node.kubernetes.io/worker.version: v0.10.1
                    node.alpha.kubernetes.io/ttl: 0
                    nvidia.com/gpu-driver-upgrade-enabled: true
                    projectcalico.org/IPv4Address: 172.16.3.2/24
                    projectcalico.org/IPv4IPIPTunnelAddr: 192.168.251.129
                    volumes.kubernetes.io/controller-managed-attach-detach: true
CreationTimestamp:  Wed, 05 Apr 2023 17:59:53 +0100
Taints:             <none>
Unschedulable:      false
Lease:
  HolderIdentity:  dgx
  AcquireTime:     <unset>
  RenewTime:       Wed, 05 Apr 2023 18:10:15 +0100
Conditions:
  Type                 Status  LastHeartbeatTime                 LastTransitionTime                Reason                       Message
  ----                 ------  -----------------                 ------------------                ------                       -------
  NetworkUnavailable   False   Wed, 05 Apr 2023 18:00:42 +0100   Wed, 05 Apr 2023 18:00:42 +0100   CalicoIsUp                   Calico is running on this node
  MemoryPressure       False   Wed, 05 Apr 2023 18:10:16 +0100   Wed, 05 Apr 2023 17:59:53 +0100   KubeletHasSufficientMemory   kubelet has sufficient memory available
  DiskPressure         False   Wed, 05 Apr 2023 18:10:16 +0100   Wed, 05 Apr 2023 17:59:53 +0100   KubeletHasNoDiskPressure     kubelet has no disk pressure
  PIDPressure          False   Wed, 05 Apr 2023 18:10:16 +0100   Wed, 05 Apr 2023 17:59:53 +0100   KubeletHasSufficientPID      kubelet has sufficient PID available
  Ready                True    Wed, 05 Apr 2023 18:10:16 +0100   Wed, 05 Apr 2023 18:00:25 +0100   KubeletReady                 kubelet is posting ready status. AppArmor enabled
Addresses:
  InternalIP:  172.16.3.2
  Hostname:    dgx
Capacity:
  cpu:                128
  ephemeral-storage:  1843269236Ki
  hugepages-1Gi:      0
  hugepages-2Mi:      0
  memory:             528018228Ki
  nvidia.com/gpu:     4
  pods:               110
Allocatable:
  cpu:                128
  ephemeral-storage:  1698756925085
  hugepages-1Gi:      0
  hugepages-2Mi:      0
  memory:             527915828Ki
  nvidia.com/gpu:     4
  pods:               110
System Info:
  Machine ID:                 fe86cb15be594307a11cc9847b0eb5c2
  System UUID:                21af0608-1dd2-11b2-9c02-f24e4f55ad5c
  Boot ID:                    8856cf0b-f468-4533-a116-69cb5e55cd4e
  Kernel Version:             5.4.0-146-generic
  OS Image:                   Ubuntu 20.04.6 LTS
  Operating System:           linux
  Architecture:               amd64
  Container Runtime Version:  containerd://1.6.16
  Kubelet Version:            v1.26.3
  Kube-Proxy Version:         v1.26.3
PodCIDR:                      192.168.1.0/24
PodCIDRs:                     192.168.1.0/24
Non-terminated Pods:          (10 in total)
  Namespace                   Name                                                           CPU Requests  CPU Limits  Memory Requests  Memory Limits  Age
  ---------                   ----                                                           ------------  ----------  ---------------  -------------  ---
  calico-system               calico-node-xxglb                                              0 (0%)        0 (0%)      0 (0%)           0 (0%)         10m
  calico-system               csi-node-driver-85l8g                                          0 (0%)        0 (0%)      0 (0%)           0 (0%)         10m
  gpu-operator                gpu-feature-discovery-66tqt                                    0 (0%)        0 (0%)      0 (0%)           0 (0%)         3m41s
  gpu-operator                gpu-operator-1680714348-node-feature-discovery-worker-65n8z    0 (0%)        0 (0%)      0 (0%)           0 (0%)         4m28s
  gpu-operator                gpu-operator-66874759cd-rfgb6                                  200m (0%)     500m (0%)   100Mi (0%)       350Mi (0%)     4m28s
  gpu-operator                nvidia-dcgm-exporter-cxt85                                     0 (0%)        0 (0%)      0 (0%)           0 (0%)         3m42s
  gpu-operator                nvidia-device-plugin-daemonset-j2wmg                           0 (0%)        0 (0%)      0 (0%)           0 (0%)         3m42s
  gpu-operator                nvidia-mig-manager-74h6j                                       0 (0%)        0 (0%)      0 (0%)           0 (0%)         13s
  gpu-operator                nvidia-operator-validator-246wq                                0 (0%)        0 (0%)      0 (0%)           0 (0%)         3m42s
  kube-system                 kube-proxy-hjlbj                                               0 (0%)        0 (0%)      0 (0%)           0 (0%)         10m
Allocated resources:
  (Total limits may be over 100 percent, i.e., overcommitted.)
  Resource           Requests    Limits
  --------           --------    ------
  cpu                200m (0%)   500m (0%)
  memory             100Mi (0%)  350Mi (0%)
  ephemeral-storage  0 (0%)      0 (0%)
  hugepages-1Gi      0 (0%)      0 (0%)
  hugepages-2Mi      0 (0%)      0 (0%)
  nvidia.com/gpu     0           0
Events:
  Type    Reason                   Age                From             Message
  ----    ------                   ----               ----             -------
  Normal  Starting                 10m                kube-proxy       
  Normal  NodeHasSufficientMemory  10m (x8 over 10m)  kubelet          Node dgx status is now: NodeHasSufficientMemory
  Normal  RegisteredNode           10m                node-controller  Node dgx event: Registered Node dgx in Controller

```


when working get pods -A 

```
NAMESPACE          NAME                                                              READY   STATUS      RESTARTS   AGE
calico-apiserver   calico-apiserver-86f7d9859f-qd8l6                                 1/1     Running     0          58m
calico-apiserver   calico-apiserver-86f7d9859f-sb6dt                                 1/1     Running     0          58m
calico-system      calico-kube-controllers-6bb86c78b4-hhbgj                          1/1     Running     0          59m
calico-system      calico-node-l89c6                                                 1/1     Running     0          59m
calico-system      calico-node-xxglb                                                 1/1     Running     0          12m
calico-system      calico-typha-6d655d787d-2n9jf                                     1/1     Running     0          59m
calico-system      csi-node-driver-85l8g                                             2/2     Running     0          12m
calico-system      csi-node-driver-nd6d4                                             2/2     Running     0          59m
gpu-operator       gpu-feature-discovery-66tqt                                       1/1     Running     0          5m47s
gpu-operator       gpu-operator-1680714348-node-feature-discovery-master-5cbdmv29s   1/1     Running     0          6m34s
gpu-operator       gpu-operator-1680714348-node-feature-discovery-worker-65n8z       1/1     Running     0          6m34s
gpu-operator       gpu-operator-66874759cd-rfgb6                                     1/1     Running     0          6m34s
gpu-operator       nvidia-cuda-validator-zlrtf                                       0/1     Completed   0          5m33s
gpu-operator       nvidia-dcgm-exporter-cxt85                                        1/1     Running     0          5m48s
gpu-operator       nvidia-device-plugin-daemonset-j2wmg                              1/1     Running     0          5m48s
gpu-operator       nvidia-device-plugin-validator-fwqmw                              0/1     Completed   0          2m30s
gpu-operator       nvidia-mig-manager-74h6j                                          1/1     Running     0          2m19s
gpu-operator       nvidia-operator-validator-246wq                                   1/1     Running     0          5m48s
kube-system        coredns-787d4945fb-2gzjg                                          1/1     Running     0          61m
kube-system        coredns-787d4945fb-vcxs9                                          1/1     Running     0          61m
kube-system        etcd-gsrv                                                         1/1     Running     0          61m
kube-system        kube-apiserver-gsrv                                               1/1     Running     0          61m
kube-system        kube-controller-manager-gsrv                                      1/1     Running     0          61m
kube-system        kube-proxy-hjlbj                                                  1/1     Running     0          12m
kube-system        kube-proxy-zwlrb                                                  1/1     Running     0          61m
kube-system        kube-scheduler-gsrv                                               1/1     Running     0          61m
tigera-operator    tigera-operator-5d6845b496-vk27g                                  1/1     Running     0          59m

```

Note: When restarting the "node-feature-discovery-worker" pod might go into an unstable state, I ended up deleting ti so the master can allocate a new one. (but then it seem to go to a loopif Error  -> CrashLoopBackOff -> Running)

then I uninstalled the gpu operator helm chart 

```
helm uninstall --namespace gpu-operator gpu-operator-1680714348
```

and then reinstalled it 

```
helm install --wait --generate-name \
     -n gpu-operator --create-namespace \
      nvidia/gpu-operator \
      --set driver.enabled=false \
      --set toolkit.enabled=false
```

but I still keep getting the previous error 


PS.

helper pod, with this we can start a pod in a specific node (eg. gsrv or dgx) and run commands 

```
kubectl run -it --rm --restart=Never --image=busybox --overrides='{"apiVersion": "v1", "spec": {"nodeName": "gsrv"}}' network-test -- sh

```

you can pull an ubuntu container too 

```
kubectl run -it --rm --restart=Never --image=ubuntu:20.04 --overrides='{"spec": {"nodeName": "dgx"}}' network-test -- /bin/sh
```

I noticed that name resolution is intermittent


I think this is why the node feature discovery pod is failing (maybe `gsrv` (my control plane node) is not powerful enough?)

the command below was run on a ubuntu test pod spun up on the dgx I ran the command `traceroute gpu-operator-1680775130-node-feature-discovery-master.gpu-operator.svc.cluster.local` 

on a pod on gsrv and another on dgx to test if they work but they did not work all the time, So I felt the issue was intermittent. 

```
 hostname
network-test-on-dgx
# traceroute gpu-operator-1680775130-node-feature-discovery-master.gpu-operator.svc.cluster.local
traceroute: unknown host
# traceroute gpu-operator-1680775130-node-feature-discovery-master.gpu-operator.svc.cluster.local
traceroute to gpu-operator-1680775130-node-feature-discovery-master.gpu-operator.svc.cluster.local (10.106.146.91), 64 hops max
  1   172.16.3.2  0.003ms  0.001ms  0.001ms 
  2   172.16.3.1  0.891ms  0.495ms  0.411ms 
  3   164.39.255.82  7.398ms  7.132ms  6.870ms 
  4   *  *  * 
  5   *  *  * 
  6   *  *  * 
  7   * 

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
