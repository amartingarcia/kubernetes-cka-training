# 07 - Security
## 07.1 - Kubernetes Security Primivite
## 07.2 - Authentication
## 07.3 - TLS Introduction
## 07.4 - TLS Basic
## 07.5 - TLS in Kubernetes
## 07.6 - TLS in Kubernetes - Certificate creation
## 07.7 - View Certificate Details
## 07.8 - Certificates API
## 07.9 - Certificates
Certificados
Certificados para la CA
	Generar ca.key (clave privada)
		openssl genrsa -out ca.key 2048
	Generar solicitud de firma
		openssl req -new ca.key -subj "/CN=KUBERNETES-CA" -out ca.csr
	Firmamos el certificado
		openssl x509 -req -in ca.csr -signkey ca.key -out ca.crt
Certificados para usuario
	Generamos clave privada
		openssl genrsa -out admin.key 2048
	Generamos solicitud de firma
		openssl req -new -key admin.key -subj "CN=kube-admin" -out admin.csr
	Firmamos con la CA de kubernetes
		openssl x509 -req -in admin.csr -CA ca.crt -CAkey ca.key -out admin.crt
	Comprobar acceso del usuario
		curl https://kube-apiserver:6443/api/v1/pods \
			--key admin.key \
			--cert admin.crt \
			--cacert ca.crt
Datos de certificado
	openssl x509 -in ca.crt -text -noout

Crear un CertificateSigningRequest
	se visualiza en csr
	se aprueba con certificate

## 07.10 -  Kubeconfig
We have seen that the k8s API Server can be accessed:
```bash
curl https://my-kube:6443/api/v1/pods \
    --key admin.key
    --cert admin.crt
    --cacert ca.crt
```

With kubectl we can do the following:
```bash
kubectl get pods 
    --server my-kube:6443
    --client-key admin.key
    --client-certificate admin.crt
    --certificate-authority ca.crt
```

This is a tedious task, so we can use the kubeconfig file, which by default is located in the `$HOME/.kube/config`

```bash
kubectl get pods --kubeconfig config
```

```yaml
apiVersion: v1
kind: Config
# Current context
current-context: my-kube-dev@my-kube-admin 
# Cluster definition
clusters:
# Cluster DEV definition
- name: my-kube-dev                                     
  cluster:
    # Relative path
    certificate-authority: ca.crt                       
    server: https://my-kuybe-dev:6443
# Cluster PRE definition
- name: my-kube-pre                                     
  cluster:
    # Absolute path
    certificate-authority: /etc/kubernetes/pki/ca.crt
    server: https://my-kuybe-pre:6443
# Cluster PRO definition
- name: my-kube-pro                                     
  cluster:
    # Base64 certificate
    certificate-authority-data: Y2VydGlmaWNhdGVhc2RmYXNkZmFkc2Zhc2RmYWRzZmFzZGZhc2RmYXNkZmFzZGZhc2RmYWRzZmFzZGZhc2RmYXNkZmFzZGZ3cmV3MzRydDI=
    server: https://my-kuybe-pro:6443
# Link between cluster and user
contexts:                                               
- name: my-kube-dev@my-kube-admin-dev
  context:
    cluster: my-kube-dev
    user: my-kube-admin-dev
    # Set default namespace from context
    namespace: finance                                  
- name: my-kube-pre@my-kube-admin-pre
  context:
    cluster: my-kube-pre
    user: my-kube-admin-pre
    namespace: marketing                                
- name: my-kube-pro@my-kube-admin-pro
  context:
    cluster: my-kube-pro
    user: my-kube-admin-pro
    namespace: devops                                   
# Users definition
users:                                                  
# User DEV definition
- name: my-kube-admin-dev                               
  user:
    client-certificate: admin.crt
    client-key: admin.key
# User DEV definition
- name: my-kube-admin-dev                               
  user:
    client-certificate: admin.crt
    client-key: admin.key
# User PRO definition
- name: my-kube-admin-pro                               
  user:
    client-certificate: admin.crt
    client-key: admin.key
```

There are many commands to interact with the kubeconfig file:
- Help:
```
kubectl config -h
```
- Viewing the file:
```
kubectl config view
```
- Change of context:
```
kubectl config use-context my-kube-pro@my-kube-admin-pro
```

## 07.11 -  RBAC
To use RBAC, the first thing we need is a Role:
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  # Role name
  name: developer
  # Namespace to role apply
  namespace: develop
# Rules list
rules:
- apiGroups: [""]
  # Resources name. Example: pods, ConfigMap, Deployment, etc.
  resources: ["pods"]
  # Actions from resoureces. Example: list, watch, delete, get, etc.
  verbs: ["list", "get", "create", "update", "delete"]

- apiGroups: [""]
  resources: ["ConfigMap"]
  verbs: ["create"]
  # Allows you to limit the name of the resources on which you can perform actions.
  resourceNames: ["develop", "test"]
```

Next step, link role with user to the RoleBinding object:
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  # RoleBinding name
  name: devuser-developer-binding
  # Namespace to RoleBinding apply
  namespace: develop
subjects:
# User definition
- kind: User
  name: dev-user 
  apiGroup: rbac.authorization.k8s.io
# Role definition
roleRef:
  kind: Role
  # Role name definition
  name: developer
  apiGroup: rbac.authorization.k8s.io
```

We can execute the following actions:
- Get Roles
```bash
kubectl get roles
```
- Get RoleBindings
```bash
kubectl get rolebindings
```
- Obtain role details
```bash
kubectl describe role developer
```
- Obtain rolebindings details
```bash
kubectl describe rolebinding devuser-developer-binding
```
- Check access from my user
```bash
kubectl auth can-i create deployments
```
- Check access as user in namespace
```bash
kubectl auth can-i delete nodes --as dev-user --namespace test
```
- Know what authorization the cluster uses:
```bash
kubectl -n kube-system get pod kube-api -oyaml | grep -i auth
```


## 07.12 -  ClusterRole - ClusterRoleBinding
We can identify two types of levels for the 'api-groups'.
- At the level of namespace. The 'Role' and 'RoleBinding' resources are used, and applied to resources at the level of namespace (pods, replicasets, jobs, deployments, PVC, etc).
- At cluster level. The 'ClusterRole' and 'ClusterRoleBinding' resources are used and applied to resources at cluster level (nodes, PV, namespaces, etc).

A list of the resources can be obtained:
- At namespace level.
```bash
kubectl api-resources --namespaced=true
```
- At cluster level.
```bash
kubectl api-resources --namespaced=false
```

Example of the creation of `ClusterRole` y `ClusterRoleBinding`:
- ClusterRole.
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  # Role name
  name: role-admin
  # Namespace to role apply
  namespace: admin
# Rules list
rules:
- apiGroups: [""]
  # Resources name. Example: nodes, namespaces, etc.
  resources: ["nodes"]
  # Actions from resoureces. Example: list, watch, delete, get, etc.
  verbs: ["list", "get", "create", "delete"]
```

- ClusterRoleBinding.
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  # RoleBinding name
  name: cluster-admin-role-binding
  # Namespace to RoleBinding apply
  namespace: admin
subjects:
# User definition
- kind: User
  name: cluster-admin 
  apiGroup: rbac.authorization.k8s.io
# Role definition
roleRef:
  kind: ClusterRole
  # Role name definition
  name: role-admin
  apiGroup: rbac.authorization.k8s.io
```


## 07.13 - Image Security
Sometimes we may need to access images from a private registry. 
By default kubernetes points to Docker Hub's public registry.

- Docker Hub nginx pull:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
  - name: nginx
    image: nginx
```

- Another Registry:
* Secret
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: myregistrykey
data:
  .dockerconfigjson: UmVhbGx5IHJlYWxseSByZWVlZWVlZWVlZWFhYWFhYWFhYWFhYWFhYWFhYWFhYWFhYWFhYWxsbGxsbGxsbGxsbGxsbGxsbGxsbGxsbGxsbGxsbGx5eXl5eXl5eXl5eXl5eXl5eXl5eSBsbGxsbGxsbGxsbGxsbG9vb29vb29vb29vb29vb29vb29vb29vb29vb25ubm5ubm5ubm5ubm5ubm5ubm5ubm5ubmdnZ2dnZ2dnZ2dnZ2dnZ2dnZ2cgYXV0aCBrZXlzCg==
type: kubernetes.io/dockerconfigjson
```
* Pod
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
  - name: nginx
    image: nginx
  imagePullSecrets:                     # Property to indicate how you will download images from a registry
  - name: myregistrykey
```


## 07.14 -  Security Context
You can choose to configure the safety settings at pod or container level.
- Pod. Will be transferred to all containers.
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: security-context-pod
spec:
  securityContext:
    runAsUser: 1000                     # Run all container as UID 1000
    runAsGroup: 3000
  containers:
  - name: testing
    image: busybox
    command: [ "sh", "-c", "sleep 1h" ]
    securityContext:
      allowPrivilegeEscalation: false
```
- Container
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: security-context-container
spec:
  containers:
  - name: testing
    image: busybox
    command: [ "sh", "-c", "sleep 1h" ]
    securityContext:
      runAsUser: 1000                   # Run this container as UID 1000
      allowPrivilegeEscalation: false   # Dont allow privilege escalation
      capabilities:                     # Allos capabilities
        add: ["MAC_ADMIN"]
```
## 07.15 - Network policy
By default in kubernetes all traffic is allowed between pods. You can indicate by means of `Network policies`, from where a pod is reachable.
It is assigned the same as other kuberrnetes objects, through `Selectors` and `Labels`.
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: db-policy
spec:
  podSelector:
    matchLabels:
      role: db
  policyTypes:
  - Ingress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          role: api-pod
    ports:
    - protocol: TCP
      port: 3306
```