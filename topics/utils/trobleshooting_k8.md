---
layout: default
title: trobleshooting k8
# nav_order: 2 
# permalink: /topics/utils/target_execute
parent: Utilities
---

## Installing Kubernetes on DGX using Kubeadm (Updated for TAO 5.0.0)
<span style="background-color:LightGreen">
Created : 18/05/2023 <br />
Status: Draft
</span>


## DNS within the K8 cluster not working for a node.


## Setting up a new cluster 

If you experience container crash loops after the initial cluster installation or after installing a CNI, the containerd configuration provided with the package might contain incompatible configuration parameters. Consider resetting the containerd configuration with <br/> <br/>`containerd config default > /etc/containerd/config.toml` <br/><br/>as specified in this [getting-started.md](https://github.com/containerd/containerd/blob/main/docs/getting-started.md#advanced-topics) and then set the configuration parameters specified above accordingly.

```
sudo sh -c 'containerd config default > /etc/containerd/config.toml'
```

get logs for the unit containerd.

```
journalctl -xeu containerd
```

## [Forwarding IPv4 and letting iptables see bridged traffic](https://kubernetes.io/docs/setup/production-environment/container-runtimes/#forwarding-ipv4-and-letting-iptables-see-bridged-traffic)

```
cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
overlay
br_netfilter
EOF
```

```
sudo modprobe overlay
sudo modprobe br_netfilter
```

sysctl params required by setup, params persist across reboots

```
cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-iptables  = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward                 = 1
EOF
```

Apply sysctl params without reboot

```
sudo sysctl --system
```

Verify that the br_netfilter, overlay modules are loaded by running the following commands:

```
lsmod | grep br_netfilter
lsmod | grep overlay
```

Verify that the net.bridge.bridge-nf-call-iptables, net.bridge.bridge-nf-call-ip6tables, and net.ipv4.ip_forward system variables are set to 1 in your sysctl config by running the following command:
```
sysctl net.bridge.bridge-nf-call-iptables net.bridge.bridge-nf-call-ip6tables net.ipv4.ip_forward
```



## [Configuring docker with nvidia container runtime](https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/latest/install-guide.html#configuring-docker) 


This command below will automatically modify the `/etc/docker/daemon.json` file to include 

```
{
    "runtimes": {
        "nvidia": {
            "args": [],
            "path": "nvidia-container-runtime",
        }
    }
}
```

Command: `sudo nvidia-ctk runtime configure --runtime=docker`


## configuring containerd 

```
sudo nvidia-ctk runtime configure --runtime=containerd
```

## Manual configuration

As shown in this [nvidia guide](https://docs.nvidia.com/datacenter/cloud-native/gpu-operator/latest/getting-started.html#bare-metal-passthrough-with-pre-installed-drivers-and-nvidia-container-toolkit) we can manually do the steps above by modifying the `/etc/docker/daemon.json`

```
{
    "default-runtime": "nvidia",
    "runtimes": {
        "nvidia": {
            "args": [],
            "path": "nvidia-container-runtime",
            "runtimeArgs": []
        }
    }
}
```

restart the docker daemon 

```
sudo systemctl restart docker
```

and update `/etc/containerd/config.toml ` with 

```
version = 2
[plugins]
  [plugins."io.containerd.grpc.v1.cri"]
    [plugins."io.containerd.grpc.v1.cri".containerd]
      default_runtime_name = "nvidia"

      [plugins."io.containerd.grpc.v1.cri".containerd.runtimes]
        [plugins."io.containerd.grpc.v1.cri".containerd.runtimes.nvidia]
          privileged_without_host_devices = false
          runtime_engine = ""
          runtime_root = ""
          runtime_type = "io.containerd.runc.v2"
          [plugins."io.containerd.grpc.v1.cri".containerd.runtimes.nvidia.options]
            BinaryName = "/usr/bin/nvidia-container-runtime"
```

## [Configuring the systemd cgroup driver](https://kubernetes.io/docs/setup/production-environment/container-runtimes/#containerd-systemd)

To use the `systemd` cgroup driver in `/etc/containerd/config.toml` with `nvidia`, set

```
[plugins."io.containerd.grpc.v1.cri".containerd.runtimes.nvidia]
  ...
  [plugins."io.containerd.grpc.v1.cri".containerd.runtimes.nvidia.options]
    SystemdCgroup = true
```

set `SystemdCgroup = true` in runc options

```
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

```




restart containerd daemon 

```
sudo systemctl restart containerd
```



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

        [plugins."io.containerd.grpc.v1.cri".containerd.default_runtime.options]

      [plugins."io.containerd.grpc.v1.cri".containerd.runtimes]

	[plugins."io.containerd.grpc.v1.cri".containerd.runtimes.nvidia]
          privileged_without_host_devices = false
          runtime_engine = ""
          runtime_root = ""
          runtime_type = "io.containerd.runc.v2"

          [plugins."io.containerd.grpc.v1.cri".containerd.runtimes.nvidia.options]
            BinaryName = "/usr/bin/nvidia-container-runtime"
            # SystemdCgroup = true


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


### Examining coredns pods 

one of the coredns pods threw the error (but even after I disconnect extra cables the DNS problem is still there)

```
g@gsrv:~$ kubectl logs -n kube-system coredns-64897985d-8wzgw
[INFO] plugin/ready: Still waiting on: "kubernetes"
.:53
[INFO] plugin/reload: Running configuration MD5 = db32ca3650231d74073ff4cf814959a7
CoreDNS-1.8.6
linux/amd64, go1.17.1, 13a9191
```

I described the suspect pod

```
g@gsrv:~$ kubectl describe pod  -n kube-system coredns-64897985d-8wzgw
Name:                 coredns-64897985d-8wzgw
Namespace:            kube-system
Priority:             2000000000
Priority Class Name:  system-cluster-critical
Node:                 gsrv/172.16.1.19
Start Time:           Wed, 13 Sep 2023 18:56:46 +0000
Labels:               k8s-app=kube-dns
                      pod-template-hash=64897985d
Annotations:          <none>
Status:               Running
IP:                   10.88.0.4
IPs:
  IP:           10.88.0.4
  IP:           2001:4860:4860::4
Controlled By:  ReplicaSet/coredns-64897985d
Containers:
  coredns:
    Container ID:  containerd://6dab716085b8ee44e9c32b50f9d83defc3a22afa81a73fd78b66cb6bbf3e4db9
    Image:         k8s.gcr.io/coredns/coredns:v1.8.6
    Image ID:      k8s.gcr.io/coredns/coredns@sha256:5b6ec0d6de9baaf3e92d0f66cd96a25b9edbce8716f5f15dcd1a616b3abd590e
    Ports:         53/UDP, 53/TCP, 9153/TCP
    Host Ports:    0/UDP, 0/TCP, 0/TCP
    Args:
      -conf
      /etc/coredns/Corefile
    State:          Running
      Started:      Wed, 13 Sep 2023 18:56:50 +0000
    Ready:          True
    Restart Count:  0
    Limits:
      memory:  170Mi
    Requests:
      cpu:        100m
      memory:     70Mi
    Liveness:     http-get http://:8080/health delay=60s timeout=5s period=10s #success=1 #failure=5
    Readiness:    http-get http://:8181/ready delay=0s timeout=1s period=10s #success=1 #failure=3
    Environment:  <none>
    Mounts:
      /etc/coredns from config-volume (ro)
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-ljl9w (ro)
Conditions:
  Type              Status
  Initialized       True 
  Ready             True 
  ContainersReady   True 
  PodScheduled      True 
Volumes:
  config-volume:
    Type:      ConfigMap (a volume populated by a ConfigMap)
    Name:      coredns
    Optional:  false
  kube-api-access-ljl9w:
    Type:                    Projected (a volume that contains injected data from multiple sources)
    TokenExpirationSeconds:  3607
    ConfigMapName:           kube-root-ca.crt
    ConfigMapOptional:       <nil>
    DownwardAPI:             true
QoS Class:                   Burstable
Node-Selectors:              kubernetes.io/os=linux
Tolerations:                 CriticalAddonsOnly op=Exists
                             node-role.kubernetes.io/control-plane:NoSchedule
                             node-role.kubernetes.io/master:NoSchedule
                             node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                             node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
Events:                      <none>
g@gsrv:~$ 

```


### check port 

Kubernetes DNS used UDP 

```
nc -zvu 10.96.0.10  53
```

### solution 

* Install cluster. 
* Fix .toml file. 
* Restart services (daemon, contianerd, docker).
* Restart DGX. 
* Delete coredns pods (new pods will be generated by the deployment).
* Try DNs resolution wiht the DNS resolution tester pods.






###



+--------------------------------------+----------+----------+----------+-----------+-----------+-------------------------+-----------------------+
| job id                               | action   | status   | result   | epoch     | t_epoch   | message                 | date                  |
+======================================+==========+==========+==========+===========+===========+=========================+=======================+
| cc486af2-fcbf-40ef-9850-22d47591f540 | train    | Error    | STARTED  | None/None |           | Starting Training Loop. | [9/19/2023][13:42:13] |
+--------------------------------------+----------+----------+----------+-----------+-----------+-------------------------+-----------------------+
+--------------------------------------+----------+----------+----------+---------+----------------+---------------------------------+-----------------------+
| job id                               | action   | status   | result   | epoch   | t_epoch        | message                         | date                  |
+======================================+==========+==========+==========+=========+================+=================================+=======================+
| 3eb00111-bb25-4a1d-a75b-7eecc5638d3a | train    | Done     | SUCCESS  | 80/80   | 0:03:50.349809 | Training finished successfully. | [9/19/2023][13:25:51] |
+--------------------------------------+----------+----------+----------+---------+----------------+---------------------------------+-----------------------+
+--------------------------------------+----------+----------+----------+-----------+-----------+--------------------------------------------------------------------------------------------------------------------------------------------------------------------+----------------------+
| job id                               | action   | status   | result   | epoch     | t_epoch   | message                                                                                                                                                            | date                 |
+======================================+==========+==========+==========+===========+===========+====================================================================================================================================================================+======================+
| 58e1e341-048b-4095-a6af-97d7ef1c3432 | train    | Error    | FAILURE  | None/None |           | 2 root error(s) found.                                                                                                                                             | [9/19/2023][14:2:31] |
|                                      |          |          |          |           |           |   (0) Resource exhausted: OOM when allocating tensor with shape[32768,48,82] and type float on /job:localhost/replica:0/task:0/device:GPU:0 by allocator GPU_0_bfc |                      |
|                                      |          |          |          |           |           |    [[{{node block_4a_conv_2_1/Pad}}]]                                                                                                                                                                    |                      |
|                                      |          |          |          |           |           | Hint: If you want to see a list of allocated tensors when OOM happens, add report_tensor_allocations_upon_oom to RunOptions for current allocation info.           |                      |
|                                      |          |          |          |           |           |                                                                                                                                                                    |                      |
|                                      |          |          |          |           |           |    [[loss/add_46/_6517]]                                                                                                                                                                    |                      |
|                                      |          |          |          |           |           | Hint: If you want to see a list of allocated tensors when OOM happens, add report_tensor_allocations_upon_oom to RunOptions for current allocation info.           |                      |
|                                      |          |          |          |           |           |                                                                                                                                                                    |                      |
|                                      |          |          |          |           |           |   (1) Resource exhausted: OOM when allocating tensor with shape[32768,48,82] and type float on /job:localhost/replica:0/task:0/device:GPU:0 by allocator GPU_0_bfc |                      |
|                                      |          |          |          |           |           |    [[{{node block_4a_conv_2_1/Pad}}]]                                                                                                                                                                    |                      |
|                                      |          |          |          |           |           | Hint: If you want to see a list of allocated tensors when OOM happens, add report_tensor_allocations_upon_oom to RunOptions for current allocation info.           |                      |
|                                      |          |          |          |           |           |                                                                                                                                                                    |                      |
|                                      |          |          |          |           |           | 0 successful operations.                                                                                                                                           |                      |
|                                      |          |          |          |           |           | 0 derived errors ignored.                                                                                                                                          |                      |
+--------------------------------------+----------+----------+----------+-----------+-----------+--------------------------------------------------------------------------------------------------------------------------------------------------------------------+----------------------+
+--------------------------------------+----------+----------+----------+-----------+-----------+-------------------------+-----------------------+
| job id                               | action   | status   | result   | epoch     | t_epoch   | message                 | date                  |
+======================================+==========+==========+==========+===========+===========+=========================+=======================+
| b7faf182-9596-4fd1-b581-be5aa857e7b7 | train    | Error    | STARTED  | None/None |           | Starting Training Loop. | [9/19/2023][13:56:36] |
+--------------------------------------+----------+----------+----------+-----------+-----------+-------------------------+-----------------------+



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

