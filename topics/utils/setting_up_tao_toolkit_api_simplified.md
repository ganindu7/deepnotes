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


### Writeup


Setting up TAO toolkit in a kubernetes cluster is slightly different from the single node setup given in the nvidia examples. As in a Lab cluster there will be multiple things running at the same time. The following writeup is a summary of the steps needed to create a first deployment. 

#### Network certificates, TLS and kubernetes secret 

TLS certificates and Kubernetes secrets are used to secure communication between the TAO REST API service and it's clients. The TLS certificate is used to authenticate the service to the client, and the Kubernetes secret is used to store the private key for the certificate. This ensures that only authorized clients can access the service. 

in this section we will create CA private key, a .cnf file and then a CA certificate. The through a certificate signing request we will create a server certificate that will be incorporated into a kubernetes TLS secret. 


**Create the CA private key**

```
openssl genrsa -out rootCA.key 2048
```


Create a .cnf file

```
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

Create a CA Certificate

```
openssl req -x509 -new -nodes -key rootCA.key -sha256 -days 1825 -out rootCA.pem -config myorg.gnet.lan.cnf
```

**Generate a private key for the server certificate** 

```
openssl genrsa -out myorg.gnet.lan.key 2048
```

**Create a certificate signing request (CSR) for the server certificate.**

```
openssl req -new -key myorg.gnet.lan.key -out myorg.gnet.lan.csr -config myorg.gnet.lan.cnf
```

**Sign the server CSR with the rootCA key and create the server certificate**

```
openssl x509 -req -in myorg.gnet.lan.csr -CA rootCA.pem -CAkey rootCA.key -CAcreateserial -out myorg.gnet.lan.crt -days 825 -sha256 -extfile myorg.gnet.lan.cnf -extensions server_cert
```

**Incoperating the certificate and key with the kubernetes secret** 

Encode the certificate and key files

```
cat myorg.gnet.lan.crt | base64 > certificate_base64.txt
cat myorg.gnet.lan.key | base64 > private-key_base64.txt

```

**update the secret** 

Replace the values of tls.crt and tls.key with the base64-encoded strings from certificate_base64.txt and private-key_base64.txt files.

```
apiVersion: v1
kind: Secret
metadata:
  creationTimestamp: null
  name: secret-tls
  namespace: tao-gnet
type: kubernetes.io/tls
data:
  tls.crt: <content from certificate_base64.txt>
  tls.key: <content from private-key_base64.txt>
```


It can look that this:


```
apiVersion: v1
data:
  tls.crt: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUVuekNDQTRlZ0F3SUJBZ0lVVXRUeXJ0MjR2c1Q0MEdjOGpUSC95UWRTSXlzd0RRWUpLb1pJaHZjTkFRRUwKQlFBd2dac3hIekFkQmdOVkJBc01Gazl5WjJGdWFYcGhkR2x2Ym1Gc1ZXNXBkRTVoYldVeEd6QVpCZ2txaGtpRwo5dzBCQ1FFV0RFVnRZVR6QU5CZ05WQkFnTUJsTjFjbkpsCmVURVFNQTRHQTFVRUJ3d0hSRzl5YTJsdVp6RVnottheactualcertNQkFHQTFVRUNnd0pRblZqYUdWeUlFRkpNUmN3RlFZRFZRUUQKREE1aGFY2VUNlBYUHRmZnZmZTdrSFlwVC9Wb0FUbSsrSU9SR3JaaElaUy9oRUxINDVHZDUKb1lXUUk4S1ZKaTFRbTBxbHY4ejlMdUxOSjhGampsdjRjRmJINWRIK0RTdDhtU1pzUU9kYnZaSHJBRi9jdC9vSgp1L2ZvTmxJbStEYVc5bjNNMkdrRDJmczhTejk0aWlRaWRxL2Rha1FVQS8zSjh3ZEl4R3E3WmFqalBIQWFHUWM2ClZ1NU1meVFzNXJGS3I5YkZvUGhGL3JaRFlWc0czR1dyczFXM25YbXJqcDcxY1JFPQotLS0tLUVORCBDRVJUSUZJQ0FURS0tLS0tCg==
  tls.key: LS0tLS1CRUdJTiBQUklWQVRFIEtFWS0tLS0tCk1JSUV2UUlCQURBTkJna3Foa2lHOXcwQkFRRUZBQVNDQktjd2dnU2pBZ0VBQW9JQkFRRFRnTlpwMWpxVTdOamUKNFJlVWhXREdzcWhkdW93cVEwME5JYTFZM3k4RVZ0SHMxMlZyYlZoQU1TajU3d0xiVjdqZXAwbHJXOFVHZVJCWQpEODhFSjZOYthis1sn0tmYkeyw0TEpjbEJFNjlXV1FGcWpycTNwNUowCmk4MFkzbTBHNENpMzBrUklkZWVkRjZWRFdHNTkwRmxHUHhUcS95aHhSV0ZJZXRGZjdlM0RDT3NtZGhydGNJnMUpOSzdaZWI0U3FOMUFvR0FVOUpieGpjTkRDaFUwTUFZSEJmTgoyWDZnR2prVUtlTStSYzJVWjZHbDl3MTMyNitvVVNjeEprMEw0Qk9DUFQ4U3NkS0d4Ym9aYzFnNE52TVcwQXBBCmphd3o5a2RPUGIyd25Eemp4cExiYlhVVDIxbHFabzFvQzRMcGs2MWZJelJnVXZRWlRyM01ld0tmZnNNaG8rMUkKMlFLU2pSQUZyWGlFWStuUkRFQm5QWm89Ci0tLS0tRU5EIFBSSVZBVEUgS0VZLS0tLS0K
kind: Secret
metadata:
  creationTimestamp: "2023-04-25T18:02:14Z"
  name: tao-aisrv-gnet-secret
  namespace: tao-gnet
  resourceVersion: "2943549"
  uid: 65a989bc-9c15-4c3a-82f4-a5aae803497a
type: kubernetes.io/tls
```


**Apply the updated secret to the K8 cluster**

```
kubectl apply -f aisrv-gnet-secret.yaml
```

Summary:

###Create a self-signed CA:
a. Create the CA private key
b. Create a .cnf file
c. Create a CA Certificate

###Generate a server certificate signed by the self-signed CA:
a. Generate a private key for the server certificate
b. Create a certificate signing request (CSR) for the server certificate
c. Sign the server CSR with the rootCA key and create the server certificate

###Incorporate the certificate and key with the Kubernetes secret:
a. Encode the certificate and key files
b. Update the secret
c. Apply the updated secret to the K8 cluster


#### Ingress Controller and Ingress class 

When a cluster has more than one ingress controller, it is necessary to use a custom ingress controller and an ingress class pair to control which ingress controller handles requests for a given path. This is because each ingress controller can have its own configuration and capabilities, and it is important to be able to select the ingress controller that is best suited for a given request.

By using a custom ingress controller and an ingress class pair, it is possible to control which ingress controller handles requests for a given path. This allows for more flexibility and control over how requests are handled in a cluster with multiple ingress controllers.


## install ingress-ngnix (not that the tao-gnet interface has to be present)

Create an ingress namespace

```
kubectl create namespace tao-gnet-ingress
```



*tao-ingress-class.yaml*
```
apiVersion: networking.k8s.io/v1
kind: IngressClass
metadata:
  name: tao-gnet-ingress
spec:
  controller: k8s.io/ingress-nginx

```

run `kubectl apply -f tao-ingress-class.yaml` to create the custom ingress class `tao-gnet-ingress` so we can have multiple ingress controllers

```
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
helm repo update
```

Note: the name "ingress-nginx-controller" and the namespace being the same as the tao toolkit is important because the validating webhook configuration
is set to `https://ingress-nginx-controller-admission.tao-gnet.svc:443`

```
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

The tao toolkit needs some persistent volume.

below is an example of a candidate persistent volume, tha tis locally mounted. 
Note that the `accessModes` is set to `ReadWriteOnce`

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

the accompanying values.yaml will look like (remember to match the `storageclass` of the pvc to the one that is provided by the pv, if storage class is not specified the default storage class will be used, if in doubt open the pvc in `describe` or `edit` mode to verify, AFIK `pv`s can;t be changed on the fly. so you'll have to delete and re create the `pv` if needed)

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

# Optional MLOPS setting for Weights And Biases
wandbApiKey: local-9a024cn0TthEap1kEy3e3ae706

# Optional MLOPS setting for ClearML
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


 Note: the default `storageclass` provided by the nfs-provisoner is `nfs-client` when `pvc`s do not a specify a `storageclass` it is assumed that they are using the default `storageclass`


 the command below will make `nfs-client` your default storage class'

 ```
 kubectl patch storageclass nfs-client -p '{"metadata": {"annotations":{"storageclass.kubernetes.io/is-default-class":"true"}}}'
 ```  

 you can verify this with a test pv and a pvc!


datailed writeup 

In this case rather than manually spinning up pvs we have opted to using a storage provisioner 
here my server ip is `172.16.1.22` and the mount path is `/k8pv1`

```
 helm upgrade --install -n k8-storage --create-namespace nfs-subdir-external-provisioner nfs-subdir-external-provisioner/nfs-subdir-external-provisioner --set nfs.server=172.16.1.22 --set nfs.path=/k8pv1
```

this can be done with a `values.yaml` instead explicit parameters too. 

```
nfs:
  server: 172.16.1.22
  path: /k8pv1
```



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


to make the toolkit work with our custom ingress we had to add the following line to our `values.yaml` 

```
ingress_class: "tao-gnet-ingress"
```

What this mean is for our endpoints we are directing the ingress to use the custom ingress class we created (which will then use our ingress controller and ingress class).
however this change needs to flow downstream to our ingress templates in the `templates` directory. for that we had to modify the `ingress.class field` in `ingress.yaml` and `ingress-auth.yaml` with 

```
kubernetes.io/ingress.class: {{ .Values.ingress_class }}
```

the modified ingress.yaml now looks like 

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

