---
layout: default
title: simplified TAO API setup
nav_order: 2 
# permalink: /topics/utils/target_execute
parent: Utilities
---

## Setting up the TAO API on K8 
<span style="background-color:LightGreen">
Created : 18/05/2032 | on Linux dgx 5.4.0-144-generic #161-Ubuntu SMP Fri Feb 3 14:49:04 UTC 2023 x86_64 x86_64 x86_64 GNU/Linux <br />
Status: Draft
</span>


Sure, here is the proofread and improved version of your document:

---
# Deploying the TAO API on Kubernetes (K8s)

<div style="background-color:LightGreen">
Document Created: 18/05/2032 | Environment: Linux dgx 5.4.0-144-generic #161-Ubuntu SMP Fri Feb 3 14:49:04 UTC 2023 x86_64 x86_64 x86_64 GNU/Linux<br />
Document Status: Draft
</div>

## Introduction

Deploying the TAO toolkit on a Kubernetes cluster presents some differences compared to a single node setup, as shown in the NVIDIA examples. Within a lab cluster, there will be multiple processes running simultaneously. This document outlines the steps required to create a primary deployment. 

## Network Certificates, TLS, and Kubernetes Secret

TLS certificates and Kubernetes secrets ensure secure communication between the TAO REST API service and its clients. The TLS certificate authenticates the service to the client, while the Kubernetes secret safeguards the private key for the certificate, allowing only authorized clients to access the service.

In this section, we will create a CA private key, a .cnf file, and a CA certificate. Through a Certificate Signing Request (CSR), we will generate a server certificate that will be incorporated into a Kubernetes TLS secret.

### Creating the CA Private Key

Run the following command:

```bash
openssl genrsa -out rootCA.key 2048
```

### Creating a .cnf File

Input the following content into a new .cnf file:

```bash
[ req ]
default_bits        = 2048
default_keyfile     = rootCA.key
distinguished_name  = req_distinguished_name
prompt              = no
x509_extensions     = x509_ext

[ req_distinguished_name ]
countryName                     = CountryName
stateOrProvinceName             = StateOrProvinceName
localityName                    = LocalityName
organizationName                = OrganizationName
organizationalUnitName          = OrganizationalUnitName
commonName                      = CA Common Name
emailAddress                    = EmailAddress

[ x509_ext ]
basicConstraints                = CA:true
keyUsage                        = keyCertSign, cRLSign

[ req ]
default_bits        = 2048
default_keyfile     = private.key
distinguished_name  = req_distinguished_name
prompt              = no

[ req_distinguished_name ]
countryName                     = UK
stateOrProvinceName             = Surrey
localityName                    = MyOrg
organizationName                = Bucher AI
commonName                      = myorg.gnet.lan

[ ca ]
default_ca = CA_default


[ CA_default ]
dir              = ./demoCA
certs            = $dir/certs
crl_dir          = $dir/crl
new_certs_dir    = $dir/newcerts
database         = $dir/index.txt
serial           = $dir/serial
RANDFILE         = $dir/private/.rand

private_key      = ./myCA.key
certificate      = ./myCA.pem

x509_extensions  = x509_ext

name_opt         = ca_default
cert_opt         = ca_default
default_crl_days = 30
default_md       = sha256
preserve         = no
policy           = policy_match

[ policy_match ]
countryName            = match
stateOrProvinceName    = match
organizationName       = match
organizationalUnitName = optional
commonName             = supplied
emailAddress           = optional

[ x509_ext ]
basicConstraints        = CA:true
keyUsage                = keyCertSign, cRLSign

[ server_cert ]
basicConstraints        = CA:FALSE
keyUsage                = digitalSignature, keyEncipherment
extendedKeyUsage        = serverAuth
subjectAltName          = @alt_names

[ alt_names ]
DNS.1 = myorg.gnet.lan
DNS.2 = ingress.local
DNS.3 = node2.gnet.lan
DNS.4 = name3.gsrv.lan
DNS.5 = name4.gnet.lan
IP.1 = 172.16.1.20
IP.2 = 172.16.3.2
IP.3 = 172.16.1.19
```

### Creating a CA Certificate

To create a CA certificate, run:

```bash
openssl req -x509 -new -nodes -key rootCA.key -sha256 -days 1825 -out rootCA.pem -config myorg.gnet.lan.cnf
```

### Generating a Private Key for the Server Certificate

Execute the command:

```bash
openssl genrsa -out myorg.gnet.lan.key 2048
```

### Creating a Certificate Signing Request (CSR) for the Server Certificate

To create a CSR for the server certificate, execute:

```bash
openssl req -new -key myorg.gnet.lan.key -out myorg.gnet.lan.csr -config myorg.gnet.lan.cnf
```

### Signing the Server CSR with the rootCA Key and Creating the Server Certificate

To sign the server CSR and create the server certificate, run:

```bash
openssl x509 -req -in myorg.gnet.lan.csr -CA rootCA.pem -CAkey rootCA.key -CAcreateserial -out myorg.gnet.lan.crt -days 825 -sha256 -extfile myorg.gnet.lan.cnf -extensions server_cert
```

### Incorporating the Certificate and Key with the Kubernetes Secret

Encode the certificate and key files with these commands:

```bash
cat myorg.gnet.lan.crt | base64 > certificate_base64.txt
cat myorg.gnet.lan.key | base64 > private-key_base64.txt
```


**Updating the Secret**

Replace the values of `tls.crt` and `tls.key` with the base64-encoded strings extracted from the `certificate_base64.txt` and `private-key_base64.txt` files, respectively.

```yaml
apiVersion: v1
kind: Secret
metadata:
  creationTimestamp: null
  name: secret-tls
  namespace: tao-gnet
type: kubernetes.io/tls
data:
  tls.crt: <Content from certificate_base64.txt>
  tls.key: <Content from private-key_base64.txt>
```

For example:

```yaml
apiVersion: v1
data:
  tls.crt: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0t...
  tls.key: LS0tLS1CRUdJTiBQUklWQVRFIEtFWS0tLS0t...
kind: Secret
metadata:
  creationTimestamp: "2023-04-25T18:02:14Z"
  name: tao-aisrv-gnet-secret
  namespace: tao-gnet
  resourceVersion: "2943549"
  uid: 65a989bc-9c15-4c3a-82f4-a5aae803497a
type: kubernetes.io/tls
```

**Apply the Updated Secret to the K8 Cluster**

```bash
kubectl apply -f aisrv-gnet-secret.yaml
```

**Summary:**

1. Create a self-signed CA:
   - Create the CA private key
   - Create a .cnf file
   - Create a CA Certificate
2. Generate a server certificate signed by the self-signed CA:
   - Generate a private key for the server certificate
   - Create a certificate signing request (CSR) for the server certificate
   - Sign the server CSR with the rootCA key and create the server certificate
3. Incorporate the certificate and key with the Kubernetes secret:
   - Encode the certificate and key files
   - Update the secret
   - Apply the updated secret to the K8 cluster


#### Ingress Controller and Ingress Class 

In a Kubernetes cluster with multiple Ingress Controllers, the use of a custom Ingress Controller and an Ingress Class pair is necessary. This allows precise control over which Ingress Controller manages requests for specific paths. Each Ingress Controller may have unique configurations and capabilities, so it's essential to select the appropriate one for a given request.

By using a custom Ingress Controller and Ingress Class pair, one can flexibly control which Ingress Controller manages specific path requests, enhancing cluster management with multiple Ingress Controllers.


## Installing Ingress-NGINX (Note: The `tao-gnet` namespace must be present)

First, create an Ingress namespace:

```bash
kubectl create namespace tao-gnet-ingress
```

Next, prepare a YAML configuration file for the Ingress class:

```yaml
apiVersion: networking.k8s.io/v1
kind: IngressClass
metadata:
  name: tao-gnet-ingress
spec:
  controller: k8s.io/ingress-nginx
```

Apply the YAML file to create the custom Ingress class `tao-gnet-ingress`:

```bash
kubectl apply -f tao-ingress-class.yaml
```

Update your Helm repositories and install the Ingress-Nginx:


Note: the name "ingress-nginx-controller" and the namespace being the same as the tao toolkit is important because the validating webhook configuration
is set to `https://ingress-nginx-controller-admission.tao-gnet.svc:443`

```bash
helm install ingress-nginx ingress-nginx/ingress-nginx --namespace tao-gnet --set controller.ingressClass=tao-gnet-ingress
helm install ingress-nginx ingress-nginx/ingress-nginx --namespace tao-gnet --values values.yaml
helm upgrade  ingress-nginx ingress-nginx/ingress-nginx --namespace tao-gnet --values values.yaml
```

if you use `values.yaml`

*values.yaml*
```
controller:
  ingressClass: tao-get-ingress

```
or

```
controller:
  logLevel: 2
  ingressClass: tao-gnet-ingress
  defaultTLS:
    secret: tao-gnet/tao-aisrv-gnet-secret
```

Note: That the ingress controller needs to be configured by our ingress class and the TLS secret. 


**Troubleshooting**

to examine validating webhooks for the ingress controller

```
kubectl get validatingwebhookconfiguration
```

then run the following command to check the webhook  urls

```
kubectl get validatingwebhookconfiguration tao-gnet-ingress-controller-ingress-nginx-admission -o yaml
```

get ingress classes

```
kubectl get ingressclasses
```



#### Persistent Volume (Manual)

The TAO Toolkit requires some persistent volume.

Below is an example of a candidate persistent volume that is locally mounted. 
Note that the `accessModes` is set to `ReadWriteOnce`.

```
apiVersion: v1
kind: PersistentVolume
metadata:
  name: my-local-tao-pv
spec:
  storageClassName: local-storage-class
  capacity:
    storage: 200Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: /mnt/k8-local-storage-path
  nodeAffinity:
    required:
      nodeSelectorTerms:
        - matchExpressions:
            - key: kubernetes.io/hostname
              operator: In
              values:
                - node2
  persistentVolumeReclaimPolicy: Retain
  volumeMode: Filesystem                              
```

The accompanying values.yaml will look as follows. Remember to match the `storageclass` of the PersistentVolumeClaim (PVC) to the one provided by the PersistentVolume (PV). If a storage class is not specified, the default storage class will be used. If in doubt, open the PVC in `describe` or `edit` mode to verify. As far as I know, PVs can't be changed on the fly. You'll have to delete and recreate the PV if necessary.

```
# TAO Toolkit API container info
image: nvcr.io/nvidia/tao/tao-toolkit:4.0.0-api
imagePullSecret: imagepullsecret
imagePullPolicy: Always 

# Optional HTTPS settings for ingress controller
host: aisrv.gnet.lan
tlsSecret: tao-aisrv-gnet-secret

ingress_class: "tao-gnet-ingress"

storageAccessMode: ReadWriteOnce
storageSize: 100Gi
ephemeral-storage: 8Gi
limits.ephemeral-storage: 50Gi
requests.ephemeral-storage: 10Gi

# Optional NVIDIA Starfleet authentication
#authClientId: bnSePYullXlG-504nOZn0pEDhoCdYR8ysm088w

# Starting TAO Toolkit jobs info
backend: local-k8s
numGpus: 4
imageTf: nvcr.io/nvidia/tao/tao-toolkit:4.0.0-tf1.15.5
imagePyt: nvcr.io/nvidia/tao/tao-toolkit:4.0.0-pyt
imageDnv2: nvcr.io/nvidia/tao/tao-toolkit:4.0.0-tf1.15.5
imageDefault: nvcr.io/nvidia/tao/tao-toolkit:4.0.0-tf1.15.5

# To opt out of providing anonymous telemetry data to NVIDIA
telemetryOptOut: no

# Optional MLOps settings for Weights And Biases
wandbApiKey: local-9a024cn0TthEap1kEy3e3ae706

# Optional MLOps settings for ClearML
clearMlWebHost: http://clearml.gnet.lan:30080
clearMlApiHost: http://clearml.gnet.lan:30008
clearMlFilesHost: http://clearml.gnet.lan:30081
clearMlApiAccessKey: PQ7X90N0tTheKeyF4N4
clearMlApiSecretKey: 6LVrMvSn0TthesEcre7nHUv

```


#### Persistent Volume (Through Storage Provisioner)


TL;DR

```console
$ helm repo add nfs-subdir-external-provisioner https://kubernetes-sigs.github.io/nfs-subdir-external-provisioner/
$ helm install nfs-subdir-external-provisioner nfs-subdir-external-provisioner/nfs-subdir-external-provisioner \
    --set nfs.server=x.x.x.x \
    --set nfs.path=/exported/path
```


Note: The default storageclass provided by the nfs-provisioner is nfs-client. When PersistentVolumeClaims (PVCs) do not specify a storageclass, it is assumed that they are using the default storageclass.

The following command will set nfs-client as your default storage class:

 ```
 kubectl patch storageclass nfs-client -p '{"metadata": {"annotations":{"storageclass.kubernetes.io/is-default-class":"true"}}}'
 ```  

You can verify the proper setup with a test PV and PVC!

Detailed Walkthrough
Rather than manually creating Persistent Volumes (PVs), we opt for using a storage provisioner. Here, my server IP is 172.16.1.22 and the mount path is /k8pv1

```
 helm upgrade --install -n k8-storage --create-namespace nfs-subdir-external-provisioner nfs-subdir-external-provisioner/nfs-subdir-external-provisioner --set nfs.server=172.16.1.22 --set nfs.path=/k8pv1
```

You can also use a values.yaml file instead of explicit parameters:


```
nfs:
  server: 172.16.1.22
  path: /k8pv1
```

The following values.yaml file shows how you can configure the TAO Toolkit API container, optional HTTPS settings for the ingress controller, optional NVIDIA Starfleet authentication, starting TAO Toolkit jobs info, and optional MLOps settings for Weights and Biases and ClearML:

```
# TAO Toolkit API container info
image: nvcr.io/nvidia/tao/tao-toolkit:4.0.0-api
imagePullSecret: imagepullsecret
imagePullPolicy: Always 

# Optional HTTPS settings for ingress controller
host: aisrv.gnet.lan
tlsSecret: tao-aisrv-gnet-secret

ingress_class: "tao-gnet-ingress"

storageAccessMode: ReadWriteMany
storageSize: 100Gi
ephemeral-storage: 8Gi
limits.ephemeral-storage: 50Gi
requests.ephemeral-storage: 10Gi

# Optional NVIDIA Starfleet authentication
#authClientId: bnSePYullXlG-504nOZn0pEDhoCdYR8ysm088w

# Starting TAO Toolkit jobs info
backend: local-k8s
numGpus: 4
imageTf: nvcr.io/nvidia/tao/tao-toolkit:4.0.0-tf1.15.5
imagePyt: nvcr.io/nvidia/tao/tao-toolkit:4.0.0-pyt
imageDnv2: nvcr.io/nvidia/tao/tao-toolkit:4.0.0-tf1.15.5
imageDefault: nvcr.io/nvidia/tao/tao-toolkit:4.0.0-tf1.15.5

# To opt out of providing anonymous telemetry data to NVIDIA
telemetryOptOut: no

# Optional MLOPS setting for Weights And Biases
wandbApiKey: local-9a024cn0TthEap1kEy3e3ae706

# Optional MLOPS setting for ClearML
clearMlWebHost: http://clearml.gnet.lan:30080
clearMlApiHost: http://clearml.gnet.lan:30008
clearMlFilesHost: http://clearml.gnet.lan:30081
clearMlApiAccessKey: PQ7X90N0tTheKeyF4N4
clearMlApiSecretKey: 6LVrMvSn0TthesEcre7nHUv

```


#### Changes to the `values.yaml` and ingress templates


To configure the toolkit to work with our custom ingress, we had to introduce the following line to our values.yaml file:

```
ingress_class: "tao-gnet-ingress"
```

This line instructs our endpoints to use the custom ingress class we've created. Consequently, our ingress controller and ingress class are being employed. This change, however, must be propagated to our ingress templates located in the templates directory. As a result, we had to modify the ingress.class field in ingress.yaml and ingress-auth.yaml as follows: 

```
kubernetes.io/ingress.class: {{ .Values.ingress_class }}
```

The updated ingress.yaml now looks as follows:
 

```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: {{ .Release.Name }}-ingress
  namespace: {{ .Release.Namespace }}
  annotations:
    kubernetes.io/ingress.class: {{ .Values.ingress_class }}
    nginx.ingress.kubernetes.io/use-regex: "true"
    nginx.ingress.kubernetes.io/rewrite-target: /api/v1/login/$2
{{- if .Values.corsOrigin }}
    nginx.ingress.kubernetes.io/enable-cors: "true"
    nginx.ingress.kubernetes.io/cors-allow-origin: {{ .Values.corsOrigin }}
{{- end }}
spec:
{{- if .Values.tlsSecret }}
  tls:
  - secretName: {{ .Values.tlsSecret }}
{{- if .Values.host }}
    hosts:
    - {{ .Values.host }}
{{- end }}
{{- end }}
  rules:
  - http:
      paths:
      - path: /{{ .Release.Namespace }}/api/v1/login(/|$)(.*)
        pathType: Prefix
        backend:
          service:
            name: {{ .Release.Name }}-service
            port:
              number: 8000
{{- if eq .Release.Namespace "default" }}
      - path: /api/v1/login(/|$)(.*)
        pathType: Prefix
        backend:
          service:
            name: {{ .Release.Name }}-service
            port:
              number: 8000
{{- end }}
{{- if and .Values.host }}
    host: {{ .Values.host }}
{{- end }}
```

`ingress-auth.yaml`

```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: {{ .Release.Name }}-ingress-auth
  namespace: {{ .Release.Namespace }}
  annotations:
    kubernetes.io/ingress.class: {{ .Values.ingress_class }}
    nginx.ingress.kubernetes.io/use-regex: "true"
    nginx.ingress.kubernetes.io/rewrite-target: /api/v1/user/$2
    nginx.ingress.kubernetes.io/auth-url: http://{{ .Release.Name }}-service.{{ .Release.Namespace }}.svc.cluster.local:8000/api/v1/auth
    nginx.ingress.kubernetes.io/client-max-body-size: 0m
    nginx.ingress.kubernetes.io/proxy-body-size: 0m
    nginx.ingress.kubernetes.io/body-size: 0m
    nginx.ingress.kubernetes.io/client-body-buffer-size: 50m
    nginx.ingress.kubernetes.io/proxy-buffer-size: 8k
    nginx.ingress.kubernetes.io/proxy-connect-timeout: "3600"
    nginx.ingress.kubernetes.io/proxy-read-timeout: "3600"
    nginx.ingress.kubernetes.io/proxy-send-timeout: "3600"
{{- if .Values.corsOrigin }}
    nginx.ingress.kubernetes.io/enable-cors: "true"
    nginx.ingress.kubernetes.io/cors-allow-origin: {{ .Values.corsOrigin }}
    nginx.ingress.kubernetes.io/cors-expose-headers: "X-Pagination-Total"
{{- end }}
spec:
{{- if .Values.tlsSecret }}
  tls:
  - secretName: {{ .Values.tlsSecret }}
{{- if .Values.host }}
    hosts:
    - {{ .Values.host }}
{{- end }}
{{- end }}
  rules:
  - http:
      paths:
      - path: /{{ .Release.Namespace }}/api/v1/user(/|$)(.*)
        pathType: Prefix
        backend:
          service:
            name: {{ .Release.Name }}-service
            port:
              number: 8000
{{- if eq .Release.Namespace "default" }}
      - path: /api/v1/user(/|$)(.*)
        pathType: Prefix
        backend:
          service:
            name: {{ .Release.Name }}-service
            port:
              number: 8000
{{- end }}
```

you can check if these templates render correctly with the command 

```
helm template my-release mychart --values values.yaml --output-dir rendered
```


**Troubleshooting** 

Make sure the created ingresses have the correct ingress class. if they don't you can fix that by editing or with a `helm upgrade command` even when installing the tao-toolkit api try to use the `helm upgrade --install pattern` (this I have not tested yet)   





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

