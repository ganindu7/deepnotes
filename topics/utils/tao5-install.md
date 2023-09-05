---
layout: default
title: tao5 install notes 
# nav_order: 2 
# permalink: /topics/utils/target_execute
parent: Utilities
---

## Installilling TAO5 
<span style="background-color:LightGreen">
Created : 26/05/2022 <br />
Status: Draft
</span>


### Writeup

Checklist 

* k8 secret is created 
* storage provisionser is created and default storage class is set to the storage class of the helm chart (e.g. nfs-client) 
* ingress controller is installed 


my instll feedback 


```
helm upgrade  --install  tao-toolkit-api tao-toolkit-api/  --values tao-toolkit-api/values.yaml -n tao-gnet 
Release "tao-toolkit-api" does not exist. Installing it now.
W0728 15:27:40.208231  302145 warnings.go:70] path /tao-gnet/openapi.json cannot be used with pathType Prefix
W0728 15:27:40.208223  302145 warnings.go:70] path /tao-gnet/openapi.yaml cannot be used with pathType Prefix
W0728 15:27:40.208231  302145 warnings.go:70] path /tao-gnet/api/v1/user(/|$)(.*) cannot be used with pathType Prefix
W0728 15:27:40.208236  302145 warnings.go:70] path /tao-gnet/api/v1/login(/|$)(.*) cannot be used with pathType Prefix
NAME: tao-toolkit-api
LAST DEPLOYED: Fri Jul 28 15:27:39 2023
NAMESPACE: tao-gnet
STATUS: deployed
REVISION: 1
TEST SUITE: None

```

I got following warnings (from the second install)

```
Release "tao-toolkit-api" does not exist. Installing it now.
W0808 08:29:30.920884 2848700 warnings.go:70] path /tao-gnet/api/v1/user(/|$)(.*) cannot be used with pathType Prefix
W0808 08:29:30.920962 2848700 warnings.go:70] path /tao-gnet/openapi.yaml cannot be used with pathType Prefix
W0808 08:29:30.953083 2848700 warnings.go:70] path /tao-gnet/api/v1/login(/|$)(.*) cannot be used with pathType Prefix
W0808 08:29:30.954079 2848700 warnings.go:70] path /tao-gnet/openapi.json cannot be used with pathType Prefix
NAME: tao-toolkit-api
LAST DEPLOYED: Tue Aug  8 08:29:30 2023
NAMESPACE: tao-gnet
STATUS: deployed
REVISION: 1
TEST SUITE: None
```

check for various probes 


liveness probe:

Exec into the pod (`e.g. kubectl exec -it -n tao-gnet tao-toolkit-api-app-pod-5ffc48cd57-nc2qp -- /bin/bash`)

```
curl -X GET "http://localhost:8000/api/v1/health/liveness"
```

readiness 

```
curl -X GET "http://localhost:8000/api/v1/health/readiness
```

-------------------
Errors:

Whenever I restart the k8 master The API login fails by timing out:  I keep getting this in error logs

potential fix 

use the stable version of ingress nginx 

helm uninstall ingress-nginx -n tao-gnet
helm repo list
remove ingress-nginx 
helm repo add nginx-stable https://helm.nginx.com/stable
helm install ingress-nginx nginx-stable/nginx-ingress -n tao-gnet


-----------------
TAO files: 

[TAO Notebooks](https://catalog.ngc.nvidia.com/orgs/nvidia/teams/tao/resources/tao-getting-started/files)



-----------------


```
Unauthorized: Credentials error: HTTPSConnectionPool(host='authn.nvidia.com', port=443): Max retries exceeded with url: /token?service=ngc (Caused by NewConnectionError('<urllib3.connection.HTTPSConnection object at 0x7f9605168a90>: Failed to establish a new connection: [Errno -3] Temporary failure in name resolution'))
Wed Aug  9 11:10:55 2023 - SIGPIPE: writing to a closed pipe/socket/fd (probably the client disconnected) on request /api/v1/login/LOL (ip 192.168.251.163) !!!
Wed Aug  9 11:10:55 2023 - uwsgi_response_writev_headers_and_body_do(): Broken pipe [core/writer.c line 306] during GET /api/v1/login/LOL (192.168.251.163)
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