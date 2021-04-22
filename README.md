# k8s-cka-certified
## Tabla de contenidos
- [Tabla de contenidos](#tabla-de-contenidos)
- [01.Introduccion](#01---introduccion)
    - [01.1.Detalles de la certificación](#011---detalles-de-la-certificación)
- [02.Core Concepts](#02---conceptos-principales)
- [03.Scheduling](#03---scheduling)
- [04.Logging & monitoring](#04---logging-monitoring)
- [Application Lifecycle Management](#05---application-lifecycle-management)
- [Cluster maintenance](#06---cluster-maintenance)
- [Security](#07---security)
- [Storage](#08---storage)
- [Networking](#09---networking)
- [Install kubernetes hard way](#10---install-kubernetes-hard-way)
- [install kubernetes the kubeadm](#11---install-kubernetes-the-kubeadm)
- [End to End test on a Kubernetes cluster](#12---end-to-end-test-on-a-kubernetes-cluster)
- [Troubleshooting](#13---troubleshooting)
- [Other Topics](#14---other-topics)

## 01 - Introducción
La finalidad de este repositorio es tener una guia para obtener la certificación de Kubernetes __CKA__.

Se divide en directorios, que contienen ficheros README.md con la información necesaria para poder pasar el examen de certificación.

Introducción al curso:
* El examen se puede comprar en la siguiente web: https://www.cncf.io/certification/cka/
* En caso de examen fallido, podrá presentarse de nuevo sin coste, en un plazo máximo de 12 meses.
* No es un examen convencional, sino ejercicios que ponen a prueba sus conocimientos.
* Podrá consultar la documentación oficial de kubernetes en el examen

### 01.1 - Detalles de la certificación
Algunas referencias al examen:
* Administrador Certificado de Kubernetes: https://www.cncf.io/certification/cka/
* Plan de estudios del examen (temas): https://github.com/cncf/curriculum
* Manual del candidato: https://www.cncf.io/certification/candidate-handbook
* Consejos para el examen: http://training.linuxfoundation.org/go//Important-Tips-CKA-CKAD

Use the code - show code in Udemy - while registering for the CKA or CKAD exams at Linux Foundation to get a 15% discount.



## 02 - Conceptos principales
### 02.1 - Arquitectura del Cluster
### 02.2 - ETCD para principiantes
### 02.3 - ETCD en Kubernetes
### 02.4 - Kube API Server
### 02.5 - Kube Controller Manage
### 02.6 - Kube Scheduler
### 02.7 - Kubelet
### 02.8 - Kube-proxy
### 02.9 - Pods
### 02.10 - ReplicaSet
### 02.11 - Deployments
### 02.12 - Namespaces
### 02.13 - Services
#### 02.13.1 - Services - ClusterIP
#### 02.13.2 -Services - LoadBalance
### 02.14 - Imperativo Vs Declarativo
### 02.15 - kubectl apply (Comandos)



## 03 - Scheduling
### 03.1 - Manual Scheduling
### 03.2 - Labels and Selectors
### 03.3 - Taints and Tolerations
### 03.4 - Node Selectors
### 03.5 - Node Affinity
### 03.6 - Taints and Tolerations Vs Node Affinity 
### 03.7 - Limites y Requerimientos de recursos
### 03.8 - DaemonSets
### 03.9 - Static Pods
### 03.10 - Multiple Scheduler


## 04 - Logging and Monitoring
### 04.1 - Monitorizar componentes del Cluster
### 04.2 - Manejar logs de aplicación



## 05 - Application Lifecycle Management
### 05.1 - Rolling Updates and Rollbacks
### 05.2 - Configurar aplicciones
### 05.3 - Commands
### 05.4 - Commands and Arguments
### 05.5 - Configurar ENVs en aplicaciones
### 05.6 - Configurar Secrets en aplicaciones
### 05.7 - Escalar aplicaciones
### 05.8 - Multi-container Pods Desing Pattern
### 05.9 - Init Containers



## 06 - Cluster Maintenance
### 06.1 - OS upgrades
### 06.2 - Kubernetes Software Versions
### 06.3 - Cluster Upgrade process
### 06.4 - Backup and Restore methods
### 06.5 - Working with ETCD



# #07 - Security
### 07.1 - Kubernetes Security Primivite
### 07.2 - Authentication
### 07.3 - TLS Introduction
### 07.4 - TLS Basic
### 07.5 - TLS in Kubernetes
### 07.6 - TLS in Kubernetes - Certificate creation
### 07.7 - View Certificate Details
### 07.8 - Certificates API
### 07.9 - Certificates
1. Certificados para la CA
* Generar ca.key (clave privada):
```bash
openssl genrsa -out ca.key 2048
```

* Generar solicitud de firma:
```bash
openssl req -new ca.key -subj "/CN=KUBERNETES-CA" -out ca.csr
```

* Firmamos el certificado:
```bash
openssl x509 -req -in ca.csr -signkey ca.key -out ca.crt
```

2. Certificados para usuario
* Generamos clave privada:
```bash
openssl genrsa -out admin.key 2048
```

* Generamos solicitud de firma:
```bash
openssl req -new -key admin.key -subj "CN=kube-admin" -out admin.csr
```

* Firmamos con la CA de kubernetes:
```bash
openssl x509 -req -in admin.csr -CA ca.crt -CAkey ca.key -out admin.crt
```

3. Comprobar acceso del usuario:
```bash
curl https://kube-apiserver:6443/api/v1/pods \
	--key admin.key \
	--cert admin.crt \
    --cacert ca.crt
```

* Datos de certificado:
```bash
openssl x509 -in ca.crt -text -noout
```

* Crear un CertificateSigningRequest:
	se visualiza en csr
	se aprueba con certificate

### 07.10 -  Kubeconfig
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

### 07.11 -  RBAC
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


### 07.12 -  ClusterRole - ClusterRoleBinding
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


### 07.13 - Image Security
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


### 07.14 -  Security Context
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
### 07.15 - Network policy
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



# 08 Storage
### 08.1 - Volumes
The containers are intended to last for a short period of time.
They are requested when they are needed to die, and their data are destroyed with the container.
To ensure that the data persists, we attach volumes when they are created, preventing their loss even if the container is disposed of.
Kubernetes has the same sharing.

This configuration is not recommended when there is more than one node.
- POD with host path
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: test-pd
spec:
  containers:
  - image: alpine
    name: alpine
    command: ["/bin/sh","-c"]
    args: ["shuf -i 0-100 -n 1 >> /opt/number.out;"]
    volumeMounts:                                       # Allows a volume to be mounted
    - mountPath: /opt                                   # Indicates the mounting point in POD
      name: test-volume                                 # Name of the volume
  volumes:                                              # Declaration of volumes
  - name: test-volume                                   # Name of the volume
    hostPath:                                           # It is created on a path of the host
      path: /data                                       # Directory location on host
      type: Directory
```

- POD with AWS EBS:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: test-pd
spec:
  containers:
  - image: alpine
    name: alpine
    command: ["/bin/sh","-c"]
    args: ["shuf -i 0-100 -n 1 >> /opt/number.out;"]
    volumeMounts:                                       # Allows a volume to be mounted                      
    - mountPath: /opt                                   # Indicates the mounting point in POD                         
      name: test-volume                                 # Name of the volume                       
  volumes:                                              # Declaration of volumes              
  - name: test-volume                                   # Name of the volume                 
    awsElasticBlockStore:                               # Volume type: hostPath, awsElasticBlockStore, gcePersistentiDisk, etc                                
      volumeID: <volume-id>                               
      fsType: ext4
```


### 08.2 - Persistent Volumes
When the kernel environment is very large, it is not recommended to apply the previous configuration, it would imply creating a volume for each POD.
A POD is a set of volumes for the entire cluster. Users can select the storage of this group through volume claims.

- HostPath PV example:
```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-vol-host-path
spec:
  accessModes:                              # Access Mode: `ReadOnlyMany`, `ReadWriteOnce` and `ReadWriteMany`.
    - ReadWriteOnce
  capacity:                                 
    storage: 1Gi                            # Storage capacity
  hostPath:                                 # Host Path Volume type
    path: /opt/data
```

- AWS EBS PV example:
```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-vol1-aws
spec:
  accessModes:                              # Access Mode: `ReadOnlyMany`, `ReadWriteOnce` and `ReadWriteMany`.
    - ReadWriteOnce
  capacity:                                 
    storage: 1Gi                            # Storage capacity
  awsElasticBlockStore:                     # AWS EBS Volume type
    volumeID: <volumen-id>
    fsType: ext4
```


### 08.3 - Persistent Volume Claims
PVCs are storage claims to PVs. They are requested by users.
Kubernetes will try to assign the correct PV according to the request (capacity, access mode, volume mode, storage class, etc).
If you want to use a specific PV for a particular type of feature You can configure `selectors` and `labels`.

- PV labels/selector
```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-vol1-aws
  labels:
    name: my-pv
spec:
  accessModes:                              # Access Mode: `ReadOnlyMany`, `ReadWriteOnce` and `ReadWriteMany`.
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain     # Options:
                                            #   Retain: It will be removed by the administrator
                                            #   Delete: will be automatically removed
                                            #   Recycle: The content will be removed before it can be reused
  capacity:                                 
    storage: 1Gi                            # Storage capacity
  awsElasticBlockStore:                     # AWS EBS Volume type
    volumeID: <volumen-id>
    fsType: ext4
```

- PVC labels/selector
```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: myclaim
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 500Mi
  selector:
    matchLabels:
      name: my-pv
```

If there are no better options, a PVC can be linked to a PV with a larger size. 
There is a one-to-one relationship, so no other claim can use the remaining capacity. 
It will remain pending until there is a PV to be linked to.
- PVC
```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: myclaim
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 500Mi
```


### 08.4 - PVCs in PODs
Once you create a PVC use it in a POD definition file by specifying the PVC Claim name under persistentVolumeClaim section in the volumes section like this.
The same is true for ReplicaSets or Deployments. Add this to the pod template section of a Deployment on ReplicaSet.

```yaml
    apiVersion: v1
    kind: Pod
    metadata:
      name: mypod
    spec:
      containers:
        - name: myfrontend
          image: nginx
          volumeMounts:
          - mountPath: "/var/www/html"
            name: mypd
      volumes:
        - name: mypd
          persistentVolumeClaim:
            claimName: myclaim
```



### 08.7 - Storage Class
Storage Class, allows the creation of a proxy between kubernetes and the cloud. Let's take Google Cloud as an example.

- Static
Each time an application requires storage, it must first be manually provisioned.

```bash
gcloud beta compute disk create \
    --size 1GB
    --region us-east1
    pd-disk
```

- GCE PersistentDisk PV example:
```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-vol1-gce
spec:
  accessModes:                              # Access Mode: `ReadOnlyMany`, `ReadWriteOnce` and `ReadWriteMany`.
    - ReadWriteOnce
  capacity:                                 
    storage: 1Gi                            # Storage capacity
  gcePersistentiDisk:                       # GCE Volume type
    pdName: pd-disk                         # PD Name
    fsType: ext4
```

- Dynamic
For dynamic provisioning we created a Storage Class for GCE
```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: google-storage
provisioner: kubernetes.io/gce-pd           # Provisioner type. Example: kubernetes.io/aws-ebs, kubernetes.io/no-provisioner (local), etc
```

- PVC
```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: myclaim
spec:
  accessModes:
    - ReadWriteOnce
  storageClassName: google-storage
  resources:
    requests:
      storage: 500Mi
```

- POD
```yaml
    apiVersion: v1
    kind: Pod
    metadata:
      name: mypod
    spec:
      containers:
        - name: myfrontend
          image: nginx
          volumeMounts:
          - mountPath: "/var/www/html"
            name: mypd
      volumes:
        - name: mypd
          persistentVolumeClaim:
            claimName: myclaim
```



## 09 - Networking
### 09.1 - Prerequisite - Switching Routing
#### 09.1.1 - Switching
Dos máquinas (A y B) pueden ser conectadas mediante un switch, a través de sus interfaces _eth0_

- Para ver las interfaces del host ejecutamos:
```bash
ip link
1: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP mode DORMANT group default qlen 1000
    link/ether 9c:b6:d0:9b:5f:f3 brd ff:ff:ff:ff:ff:ff
```

- La red que tenemos es la 192.168.1.0, y las máquinas tienen asignadas las IPs 192.168.1.10 (A) y 192.168.1.11 (B). Linkamos IP con interface eth0
```bash
# Node A
ip addr add 192.168.1.10/24 dev eth0
# Node B
ip addr add 192.168.1.11/24 dev eth0
```

Ahora podremos entrar paquetes entre las máquinas A y B.

#### 09.1.2 - Routing
Queremos conectar dos redes con dos máquinas cada red, 192.168.1.0 y 192.168.2.0.
Un router permite comunicar estas dos redes. El enrutador posee una IP en cada red 192.168.1.1 y 192.168.2.1

Para que la red A pueda llegar a la red B, necesitamos añadir rutas de destino.
```bash
route

Kernel IP routing table
Destination     Gateway         GenmaskFlags Metric Ref    Use Iface

# Add new route
ip route add 192.168.2.0/24 via 192.168.1.1

# Show routes
route
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
192.168.2.0     192.168.1.1     255.255.255.0   UG    0      0        0 eth0
```

La siguiente cuestión, es comunicar el router con Internet, por ejemplo Google 172.217.194.0.
```
ip route add 172.217.194.0/24 via 192.168.2.1
```

Se le puede indicar que para las direcciones que no conozca, utilice una ruta por defecto
```
ip route add default via 192.168.2.1
```

Para las IPs no reconocidas lo mejor es añadir como destino 0.0.0.0/0


### 09.2 - Prerequisite - DNS

#### 09.2.1 - Resolución en red local
Dadas dos máquinas A y B en la misma red, comprobamos que obtienen conexión por ping a través de sus IPs.
Para no recordar la IP podemos identificar el nodo mediante un nombre, por ejemplo __db__.

- No conoce ningún host llamado _db_
```
ping db
ping: db: Name or service not known
```

- Para decirle al sistema que nombre db corresponde con una IP, podemos añadir una entrada al fichero /etc/hosts
```bash
cat /etc/hosts

192.168.1.11    db
```

- Ahora nuestro sistema si reconoce el nombre del host
```bash
ping db

PING db(192.168.1.11) 56(84) bytes of data.
64 bytes from db(192.168.1.11): icmp_seq=1 ttl=64 time=0.052 ms
64 bytes from db(192.168.1.11): icmp_seq=2 ttl=64 time=0.079 m
```

- Nuestros se fia ciegamente de lo que indiquemos en nuestro fichero hosts, pero puede ser cierto o no. Podemos engañar al sistema indicando que la máquina A es Google.
```
cat /etc/hosts

192.168.1.11    db
192.168.1.11    www.google.com
```

- Ahora nuestro sistema si reconoce el nombre del host
```bash
ping www.google.com

PING www.google.com(192.168.1.11) 56(84) bytes of data.
64 bytes from www.google.com(192.168.1.11): icmp_seq=1 ttl=64 time=0.052 ms
64 bytes from www.google.com(192.168.1.11): icmp_seq=2 ttl=64 time=0.079 m
```

#### 09.2.2 - Resolución con servidor DNS
Cuando un entorno crece y existen muchas máquinas, no es una buena forma, de mantener la conexión entre ellas.
Podemos utilizar una máquina demoninada DNS para que obtenga un listado de máquinas e IPs y desde nuestros hosts, podamos consultarle.

- En cada uno de los servidores se ubica un fichero que señala a un servidor de DNS.
```
cat /etc/resolv.conf

nameserver 192.168.1.100
```

Ya no es necesario que tengan los servidores entradas en el fichero /etc/hosts, pero esto no significa que no pueda tenerlas.
Mediante un archivo de configuración podemos indicar si queremos que resuelva primera con el fichero /etc/hosts o con el fichero /etc/resolv.conf.
Para ello la configuración se aplica en el fichero /etc/nsswitch.conf

Nuestros sistema desde este momento conoce las declaracionse en el fichero /etc/hosts y las declaraciones en el servidor de DNS. 
Pero si tratamos de llegar a una dirección que no conoce fallará.
Podemos agregar una nueva entrada a nuestro fichero /etc/resolv.conf, 8.8.8.8 DNS público alojado por Google que conoce todos los sitios webs de internet.
```
cat /etc/resolv.conf

nameserver 192.168.1.100
nameserver 8.8.8.8
```

> Una solución sería añadir una linea a nuestro servidor DNS para que todo lo desconocido lo reenvie a 8.8.8.8


#### 09.2.3 - Domain Names
Se llama nombre de dominio a la traducción de una IP. La razón de que un dominio esté separado por puntos es para agrupar.

La última parte de un nombre de dominio .org, .com, .net, etc son los dominios de nivel superior, y representan la intención del sitio web.

El resto del nombre se entiende por subdominio. Por ejemplo: www.google.com, maps.google.com, drive.google.com.

#### 09.2.4 - Search Domain
Cuando queremos acceder internamente a los servicios de una empresa, y queremos solamente utilizar el nombre de los servicios, podemos añadir a nuestro fichero:
```bash
cat /etc/resolv.conf

nameserver  192.168.1.100
search      mycompany.com   prod.mycompany.com
```

#### 09.2.5 - Record Types
¿Como se almacenan los registros DNS?

| Types |                 |                                    | Description                                                            |
|-------|-----------------|------------------------------------|------------------------------------------------------------------------|
| A     | web-server      | 192.168.1.1                        | Vinculan un nombre con una IP                                          |
| AAAA  | web-server      | 2001:asdf:dfgh:9616:316651:fgh45   | Vinculan un nombre con una IPv6                                        |
| CNAME | foo.web-server  | eat.web-server, hungry.web-server  | Contiene un __alias__ o nombre alternativo para otro registro A o AAAA |

Otros registros:
| Types |                                                                       | Description                                                            |
|-------|-----------------------------------------------------------------------|------------------------------------------------------------------------|
| SOA   | ns1.dnsimple.com admin.dnsimple.com 2013022001 86400 7200 604800 300  | _Start of Authority_ Información sobre la zona que se organiza, importantes para la trasnferencia de zonas|
| MX    | ASPMX.L.GOOGLE.COM                                                    | __Mail Exchange__, intercambio que se produce mediante un servidor SMTP, es posible configurar varios con distintas prioridades para compensar fallos |
| PTR   | 34.216.184.93.in-addr.arpa. IN PTR example.org.                       | __Reverse lookup__, el servidor DNS puede indnicar quenombre de host pertenece a una IP  |
| NS    | ns1.subname.example.com                                               | Servidor de nombres de una zona, determina donde recae la responsabilidad de una zona concreta |
| TXT   | v=spf1 ip4:192.0.2.0/24 ip4:198.51.100.123 ip6:2620:0:860::/46 a -all | Contienen texto, como información para usuarios, también pueden añadirse detalles sobre la empresa |
| SRV   | _sip._tcp.example.com.                                                | Informa sobre los servicios disponibles del dominio |
| LOC   | LOC record statdns.net.   IN LOC   52 22 23.000 N 4 53 32.000 E -2.00m 0.00m 10000m 10m | Ubicación fisica del servidor |

#### 09.2.6 - Tools for debug
* nslookup: consultar el nombre de host de un servidor DNS
```
nslookup www.google.es 
Server:		127.0.0.53
Address:	127.0.0.53#53

Non-authoritative answer:
Name:	www.google.es
Address: 74.125.193.94
Name:	www.google.es
Address: 2a00:1450:400b:c01::5e
```

* dig: comprueba la resolución de nombres.
```
dig www.google.es

; <<>> DiG 9.11.3-1ubuntu1.13-Ubuntu <<>> www.google.es
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 56212
;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 65494
;; QUESTION SECTION:
;www.google.es.			IN	A

;; ANSWER SECTION:
www.google.es.		106	IN	A	74.125.193.94

;; Query time: 51 msec
;; SERVER: 127.0.0.53#53(127.0.0.53)
;; WHEN: Tue Nov 03 15:28:07 CET 2020
;; MSG SIZE  rcvd: 58
```

### 09.3 - Prerequisite - CoreDNS
In the previous lecture we saw why you need a DNS server and how it can help manage name resolution in large environments with many hostnames and Ips and how you can configure your hosts to point to a DNS server. In this article we will see how to configure a host as a DNS server.

We are given a server dedicated as the DNS server, and a set of Ips to configure as entries in the server. There are many DNS server solutions out there, in this lecture we will focus on a particular one – CoreDNS.

So how do you get core dns? CoreDNS binaries can be downloaded from their Github releases page or as a docker image. Let’s go the traditional route. Download the binary using curl or wget. And extract it. You get the coredns executable.

Run the executable to start a DNS server. It by default listens on port 53, which is the default port for a DNS server.

Now we haven’t specified the IP to hostname mappings. For that you need to provide some configurations. There are multiple ways to do that. We will look at one. First we put all of the entries into the DNS servers /etc/hosts file.

And then we configure CoreDNS to use that file. CoreDNS loads it’s configuration from a file named Corefile. Here is a simple configuration that instructs CoreDNS to fetch the IP to hostname mappings from the file /etc/hosts. When the DNS server is run, it now picks the Ips and names from the /etc/hosts file on the server.

CoreDNS also supports other ways of configuring DNS entries through plugins. We will look at the plugin that it uses for Kubernetes in a later section.

Read more about CoreDNS here:

https://github.com/kubernetes/dns/blob/master/docs/specification.md

https://coredns.io/plugins/kubernetes/


### 09.4 - Prerequisite - Network Namespaces
Los contenedores se separan del host mediante namespaces, lo cual permite que los procesos que corren dentro, no puedan acceder a los procesos de fuera, a menos que se indique lo contrario.
Sin embargo el host, si puede ver los procesos de todos los contendores que corren en el.
```
ps aux 
USER       PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
root         1  0.1  0.0 226388  9948 ?        Ss   08:00   0:46 /sbin/init splash
root         2  0.0  0.0      0     0 ?        S    08:00   0:00 [kthreadd]
root         4  0.0  0.0      0     0 ?        I<   08:00   0:00 [kworker/0:0H]
root         6  0.0  0.0      0     0 ?        I<   08:00   0:00 [mm_percpu_wq]
root         7  0.0  0.0      0     0 ?        S    08:00   0:00 [ksoftirqd/0]
root         8  0.1  0.0      0     0 ?        I    08:00   0:44 [rcu_sched]
root         9  0.0  0.0      0     0 ?        I    08:00   0:00 [rcu_bh]
root        10  0.0  0.0      0     0 ?        S    08:00   0:00 [migration/0]
```

### 09.5 - Prerequisite - Docker Networking
### 09.6 - Prerequisite - CNI
### 09.7 - Cluster Networking

### 09.8 - Pod Networking
Kubernetes espera que:
- Cada Pod obtenga su propia IP
- Cada Pod debería poder alcanzar cualquier otro Pod dentro del mismo nodo utilizando esa dirección IP, o de otro nodo.
- Cada Pod debería poder alcanzar cualquier otro Pod dentro de otro nodo sin usar NAT.

Existen muchas soluciones que permite esto:
- weave
- flannel
- cilium
- Network Namespaces
- CNI

Kubelet es el encargado de crear contenedores en los nodos. Cuando un contenedor se crea se invoca al CNI que actua como intermediario como argumentos, indicando:
```
--cni-conf-dir=/etc/cni/net.d
--cni-bin-dir=/etc/cni/bin
```

Los estandares de CNI indican como debe añadir y eliminar un contendor a nivel de red. Una visión del script podría ser la siguiente:
```
ADD)
# Create veth pair
# Attach veth pair
# Assing IP Address
# Bring Up Interface
ip -n <namespace> link set ...

DEL)
# Delete veth pair
ip link del ...
```

La invocación del scrit podría ser algo así:
```
./net-script.sh add <container> <namespace>
```

### 09.9 - CNI in kubernetes
CNI define las responsabilidades del tiempo de ejcución del contenedor.
- Container Runtime must create network namespace
- Identify network the container must attach to
- Container Runtime to invoke Network Plugin (bridge) whtn container is ADDed
- Container Runtime to invoke Network Plugin (bridge) whtn container is DELeted.
- JSON format of the Network Configuration

El complemento CNI se configura en el servicio kubelet en cada nodo del cluster.
Si observamos el fichero de servicio de kubelet, podremos ver:
```bash
kubelet.service
...

--network-plugin=cni \\
--cni-bin-dir=/opt/cni/bin \\
--cni-conf-dir=/etc/cni/net.d \\

...
```

También es posible ver la configuración del proceso mediante:
```bash
ps -aux | grep kubelet
```

En el directorio cni/bin se ubican todos los complemtos CNI compatibles como ejecutables.
```bash
ls /opt/cni/bin

bridge dhcp flannel host-device host-local ipvlan loopback mcvlan ptp sample tunning vlan
```

En el directorio /etc/cni/net.d, se ubica la configuración a aplicar, si hubiera varios ficheros, se aplicaría en orden alfabetico.
Contiene configuraciones relacionadas con Bridge, enrutamiento y enmascaramiento NAT. También definie si la interfaz de red debe tener asignada una dirección IP, ...

```bash
ls /etc/cni/net.d

10-bridge.conf

# Show file
cat 10-bridge.conf

{
	"cniVersion": "0.2.0",
	"name": "mynet",
	"type": "bridge",
	"bridge": "cni0",
	"isGateway": true,
	"ipMasq": true,
	"ipam": {
		"type": "host-local",
		"subnet": "10.22.0.0/16",
		"routes": [
			{ "dst": "0.0.0.0/0" }
		]
	}
}
```

### 09.10 - CNI weave
La solución que inicialmente se exponía, utilizaba un enrutador para enviar el tráfico, esto funciona cuando la red es muy pequeña y simple. 
Para entornos granes con cientos de nodos, y cientos de PODs en cada nodo no es práctico, ya que es posible que la tabla de enrutamiento no adminta tantas entradas.

El complemento CNI Weave se implementa en cada nodo. Y se comunican entre si para intercambiar información sobre los nodos, la red y los PODs.
Cada agente conoce los PODs y sus IPs de los otros nodos.

Weave crea su propio puente sobre los nodos y lo nombra como Weave. Luego, asigna la dirección IP a cada red.
Un pod puede estar en varias redes, como por ejemplo en la red de Weave o en la red de docker, la ruta que toma un paquete para llegar al destino, depende de la ruta configurada en el contendor.

Weave se asegura de que todos los PODs obtengan la ruta correcta configurada para llegar al agente, cuando se envia un paquete de un pod a otro nodo, Weave intercepta el paquete e identifica que está en una red separada. Luego, encapsula el paquete en uno nuevo con nuevo origen y destino, y lo envia a traves de la red.

#### 09.10.1 - Deploy Weave
Se puede implementar como servicios o daemon en cada nodo del cluster de forma manual. O si kubernetes ya está configurado se puede implementar como __daemonset__.

```bash
kubectlapply -f "https://cloud.weave.works/k8s/net?k8s-version=$(kubectl version | base64 | tr -d '\n')"

serviceaccount/weave-net created
clusterrole.rbac.authorization.k8s.io/weave-net created
clusterrolebinding.rbac.authorization.k8s.io/weave-net created
role.rbac.authorization.k8s.io/weave-net created
rolebinding.rbac.authorization.k8s.io/weave-net created
daemonset.extensions/weave-net created
```

#### 09.10.2 - Weave peers
Si implementó su cluster con kubeadm, puede verlo en cada nodo:
```bash
kubectlget pods –n kube-system

NAME                             READY     STATUS    RESTARTS   AGE       IP            NODE      NOMINATED NODE
coredns-78fcdf6894-99khw         1/1       Running   0          19m       10.44.0.2     master    <none>
coredns-78fcdf6894-p7dpj         1/1       Running   0          19m       10.44.0.1     master    <none>
etcd-master                      1/1       Running   0          18m       172.17.0.11   master    <none>
kube-apiserver-master            1/1       Running   0          18m       172.17.0.11   master    <none>
kube-scheduler-master            1/1       Running   0          17m       172.17.0.11   master    <none>
weave-net-5gcmb                  2/2       Running   1          19m       172.17.0.30   node02    <none>
weave-net-fr9n9                  2/2       Running   1          19m       172.17.0.11   master    <none>
weave-net-mc6s2                  2/2       Running   1          19m       172.17.0.23   node01    <none>
weave-net-tbzvz                  2/2       Running   1          19m       172.17.0.52   node03    <none>
```

```
kubectllogs weave-net-5gcmb weave –n kube-system

INFO: 2019/03/03 03:41:08.643858 Command line options: map[status-addr:0.0.0.0:6782 http-addr:127.0.0.1:6784 ipalloc-range:10.32.0.0/12 name:9e:96:c8:09:bf:c4 nickname:node02 conn-limit:30 datapath:datapathdb-prefix:/weavedb/weave-net host-root:/host port:6783 docker-api: expect-npc:trueipalloc-init:consensus=4 no-dns:true]
INFO: 2019/03/03 03:41:08.643980 weave  2.2.1
INFO: 2019/03/03 03:41:08.751508 Bridge type is bridged_fastdp
INFO: 2019/03/03 03:41:08.751526 Communication between peers is unencrypted.
INFO: 2019/03/03 03:41:08.753583 Our name is 9e:96:c8:09:bf:c4(node02)
INFO: 2019/03/03 03:41:08.753615 Launch detected -using supplied peer list: [172.17.0.11 172.17.0.23 172.17.0.30 172.17.0.52]
INFO: 2019/03/03 03:41:08.753632 Checking for pre-existing addresses on weave bridge
INFO: 2019/03/03 03:41:08.756183 [allocator 9e:96:c8:09:bf:c4] No valid persisted data
INFO: 2019/03/03 03:41:08.761033 [allocator 9e:96:c8:09:bf:c4] Initialisingvia deferred consensus
INFO: 2019/03/03 03:41:08.761091 Sniffing traffic on datapath(via ODP)
INFO: 2019/03/03 03:41:08.761659 ->[172.17.0.23:6783] attempting connection
INFO: 2019/03/03 03:41:08.817477 overlay_switch->[8a:31:f6:b1:38:3f(node03)] using fastdp
INFO: 2019/03/03 03:41:08.819493 sleeve ->[172.17.0.52:6783|8a:31:f6:b1:38:3f(node03)]: Effective MTU verified at 1438
INFO: 2019/03/03 03:41:09.107287 Weave version 2.5.1 is available; please update at https://github.com/weaveworks/weave/releases/download/v2.5.1/weave
INFO: 2019/03/03 03:41:09.284907 Discovered remote MAC 8a:dd:b5:14:8f:a3 at 8a:dd:b5:14:8f:a3(node01)
INFO: 2019/03/03 03:41:09.331952 Discovered remote MAC 8a:31:f6:b1:38:3f at 8a:31:f6:b1:38:3f(node03)
INFO: 2019/03/03 03:41:09.355976 Discovered remote MAC 8a:a5:9c:d2:86:1f at 8a:31:f6:b1:38:3f(node0
```

### 09.11 - IP Address Management - Weave
Esta sección cubre la herramienta que elimina redes y los nodos a asignados a una subred.

CNI dice que el responsabilidad del proveedor se soluciones de red ocuparse de asignar IPs a los contendores.

En el fichero de configuración de CNI, tiene una sección llamada __"ipam"__ donde se indica el tipo de complemento y la ruta que utilizará.

```json
cat  /etc/cni/net.d/10-bridge.conf
{
	"cniVersion": "0.2.0",
	"name": "mynet",
	"type": "bridge",
	"bridge": "cni0",
	"isGateway": true,
	"ipMasq": true,
	"ipam": {
		"type": "host-local",
		"subnet": "10.22.0.0/16",
		"routes": [
			{ "dst": "0.0.0.0/0" }
		]
	}
}
```


### 09.12 - Service Networking
Recapitulando anteriores temas, rara vez configuraremos dos pods para comunicarse directamente entre si.
Si necesitamos que un pod acceda a servicios de otro pod, siempre utilizará un objecto SERVICE. El objecto Service se crea "delante" de un POD, y obtiene una dirección IP y un nombre, otros PODs, accederán mediante la IP o el nombre del Service.

#### 09.12.1 - Cluster IP
Cuando se crea un service es accesible desde todo el cluster, independientemente del nodo en el que se encuentren.

#### 09.12.2 - NodePort
Cuando necesitemos que un servicio se accesible desde fuera del cluster, utilizamos Service de tipo NodePort.
También se le asigna una dirección IP y un nombre y es accesible desde el cluster, pero además expone la aplicación en un puerto en todos los nodos.

#### 09.12.3 - Networking
El servicio kubelet es el encargado de crear los PODs, y observa los cambios del cluster a través del API Server.
Cada vez que se crea un POD se invoca al complemento CNI para configurar la red para ese POD.

Del mismo modo, cada nodo ejecuta otro componente kube-proxy. Kube-proxy observa los cambios en el cluster a través del API Server, y cada vez que se crea  un nuevo servicio, kube-proxy entra en acción.

Cuando se crea un service, se le asigna una IP de un rango predefinido. Los componentes de kube-proxy que se ejecutan en cada nodo, obtienen esa IP y crean las reglas de reenvio a cada nodo del cluster, indicando que cada tráfico que llegue a la IP del Service debe ir a la IP del POD.

Kube-proxy, adminte diferentes formas de crear reglas:
- Como el userspace donde kube-proxy escucha en un puerto para cada servicio y conecta las conexiones de proxy a los pods.
- Reglas ipvs, tablas IPs.
- Iptables

El valor predeterminado es __iptables__. Podría verse el valor establecido visualizando los logs de los PODs de kube-proxy.
Puedes ver las reglas ejecutando:

```bash
iptables -L -t net | grep <service-name>

KUBE-SVC-XA5OGUC7YRHOS3PU	tcp	--	anywhere 	10.103.132.104	/*	default/db-service:		cluster IP 	*/ 	tcp   dpt:3306
DNAT						tcp --	anywhere 	anywhere		/*	default/db-service:		*/ 	tcp 	to:10.244.1.2:3306
KUBE-SEP-JBWCWHHQM57V2WN7	tcp -- 	anywhere	anywhere		/*	default/db-service:		*/
```

También es posible ver, como kube-proxy crea estas reglas de entrada:
```bash
cat /var/log/kube-proxy.log

I0307 04:29:29.883941	1	server_others.go:140] Using iptables Proxier.
I0307 04:29:29.912037	1	server_others.go:174] Tearing down inactive rules.
I0307 04:29:30.027360	1	server.go:448] Version: v1.11.8
I0307 04:29:30.049773	1	conntrack.go:98] Set sysctl 'net/netfilter/nf_conntrack_max' to 131072
I0307 04:29:30.049945	1	conntrack.go:52] Setting nf_conntrack_max to 131072
I0307 04:29:30.050701	1	conntrack.go:83] Setting conntrack hashsize to 32768
I0307 04:29:30.050701	1	proxier.go:294] Adding new service “default/db-service:3306" at 10.103.132.104:3306/TCP
```


# 09.13 - DNS in kubernetes
Kubernetes integra un servicio de DNS predeterminado. Cada vez que se crea un objecto Service, se crea un registro con nombre y IP del servicio.
Por ello cualquier pod, puede llegar al servicio utilizando su nombre.

Para cada namespace, el servicio de DNS crea un subdominio. Todos los servicio se agrupan en otro subdominio __SVC__.

```bash
curl http://web-service.apps
Welcome to NGINX!

curl http://web-service.apps.svc
Welcome to NGINX!

curl http://web-service.apps.svc.cluster.local
Welcome to NGINX!
```

Con los PODs, ocurre lo mismo, sin embargo kubernetes sustituye los puntos de la IP por guiones.
![dns_pod](../img/09_dns_pod.png)


# 09.14 - CoreDNS in kubernetes
En las versionse previas a v1.12, kubernetes implementaba un servicio llamado kube-dns, desde esa versión hacia delante, el sevicio es llamado CoreDNS.

El servidor CoreDNS se implementan como POD en el namespace kube-system.

Utiliza un fichero de configuración `/etc/coredns/Corefile`, obtenido a través de un ConfigMap, y su contenido es el siguiente:
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: coredns
  namespace: kube-system
data:
  Corefile: |
    .:53 {
        errors
        health {
            lameduck 5s
        }
        ready
        kubernetes cluster.local in-addr.arpa ip6.arpa {
            pods insecure
            fallthrough in-addr.arpa ip6.arpa
            ttl 30
        }
        prometheus :9153
        forward . /etc/resolv.conf
        cache 30
        loop
        reload
        loadbalance
    }
```

Cuando se crea un POD, se registra una nueva entrada, sustituyendo los puntos por guiones, para su resolución
```bash
10-244-1-5	10.224.1.5
10-244-2-5	10.224.2.5
```

CoreDNS también despliega un SERVICE, y permite que los PODs apunten a este SERVICE en su configuración, el encargado de crear esta configuración en los PODs es __kubelet__:
```bash
cat /etc/resolv.conf

nameserver	10.96.0.10
```

Si observamos el archivo de configuración de kubelet, podremos ver la IP del servidor DNS y el dominio:
```bash
cat /var/lib/kubelet/config

...
clusterDNS:
- 10.96.0.10
clusterDomain: cluster.local
```


Desde un POD puedes obtener el FQDN de un SERVICE:
```bash
host web-service
web-service.default.svc.cluster.local has address 10.97.206.196
```

Porque en el fichero /etc/resolv.conf se indica el nombre del cluster:
```bash
cat /etc/resolv.conf

nameserver 	10.96.0.10
search		default.svc.cluster.local	  svc.cluster	 cluster.local
```

Pero esto solo ocurre con los SERVICE, para un pod, debe indicar el FQDN:
```bash
host 10-224-2-5
Host 10-224-2-5 not found: 3(NXDOMAIN)

# FQDN
host 10-224-2-5.default.pod.cluster.local
10-224-2-5.default.pod.cluster.local has address 10.224.2.5
```


# 09.15 - Ingress
Los Ingress ayudan a los usuarios a llegar a su aplicación, usando una única URL accesible externamente, que puede configurar para enrutar a diferente servicios dentro de su cluster. La ruta URL al mismo tiempo, implementa seguridad SSL, un Ingress es como un Loadl Balancer capa 7.

Aún utilizando Ingress necesita exponerlo para que sea accesible desde fuera del cluster.

Kubernetes despliega una solución compatible, como pueden ser Nginx, HAProxy o Traefik, luego configura un conjunto de reglas para configurar el Ingress. Estas soluciones se llaman __Ingress Controller__, y el conjunto de reglas __Ingress Resources__ (Objeto que desplegamos con nuestras aplicaciones).

Por defecto, nuestro cluster no viene con ningún _Ingress Controller_.

## Ingress Controller
Existen varias soluciones disponibles:
- GCP HTTP(S) Load Balancer	 (apoyado y mantenido por el proyecto Kubernetes)
- Nginx (apoyado y mantenido por el proyecto Kubernetes)
- Contour
- HAProxy
- Traefik
- Istio

Ejemplo de definición de Ingress Controller, para una versión de Nginx creada específicamnte como Ingress Controller para kubernetes.
- Necesitaremos un ConfigMap con la configuración el Ingres (puede estar vacío), en el que se configura el path para los logs, configuración SSL, etc;

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: ingress-nginx-controller
data:
  proxy-connect-timeout: "10"
  proxy-read-timeout: "120"
  proxy-send-timeout: "120"
```

- Objeto Deployment para el Ingress Controller:
```yaml
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: nginx-ingress-controller
  labels:
    app.kubernetes.io/name: ingress-nginx
spec:
  replicas: 1
  selector:
    matchLabels:
      app.kubernetes.io/name: ingress-nginx
  template:
    metadata:
      labels:
        app.kubernetes.io/name: ingress-nginx
    spec:
      containers:
	  - name: nginx-ingress-controller
	    image: quay.io/kubernetes-ingress-controller/nginx-ingress-controller:0.33.0
    
        args:
		- /nginx-ingress-controller
		- --configmap=$(POD_NAMESPACE)/nginx-configuration

		env:
          - name: POD_NAME
            valueFrom:
              fieldRef:
                fieldPath: metadata.name
          - name: POD_NAMESPACE
            valueFrom:
              fieldRef:
				fieldPath: metadata.namespace
		
		ports:
        - containerPort: 80
          hostPort: 80
        - containerPort: 443
          hostPort: 443
```

- Necesitaremos un SERVICE para exponer el Ingress Controller:
```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-ingrress
spec:
  type: NodePort
  selector:
	name: nginx-ingress
  ports:
	- name: http
	  port: 80
	  targetPort: 80
	  protocol: TCP
	- name: https
	  port: 443
	  targetPort: 443
	  protocol: TCP
```

El Ingress Controller tiene inteligencia adicional para monitorear el cluster en busca de Ingress Resources y configurar el servidor Nginx.
Para que el Ingress Controller pueda hacer todo esto, necesita una _Service Account_, con un conjunto de permisos.
```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: nginx-ingress-serviceaccount

---

apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: nginx-ingress-clusterrole
rules:
  - apiGroups:
      - ""
    resources:
      - configmaps
      - endpoints
      - nodes
      - pods
      - secrets
    verbs:
      - list
      - watch
  - apiGroups:
      - ""
    resources:
      - nodes
    verbs:
      - get
  - apiGroups:
      - ""
    resources:
      - services
    verbs:
      - get
      - list
      - update
      - watch
  - apiGroups:
      - extensions
      - "networking.k8s.io" # k8s 1.14+
    resources:
      - ingresses
    verbs:
      - get
      - list
      - watch
  - apiGroups:
      - ""
    resources:
      - events
    verbs:
      - create
      - patch
  - apiGroups:
      - extensions
      - "networking.k8s.io" # k8s 1.14+
    resources:
      - ingresses/status
    verbs:
      - update
  - apiGroups:
      - "networking.k8s.io" # k8s 1.14+
    resources:
      - ingressclasses
    verbs:
      - get
      - list
	  - watch
	  
---

apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: nginx-controller-clusterrolebinding
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: nginx-ingress-clusterrole
subjects:
  - kind: ServiceAccount
    name: nginx-ingress-serviceaccount
```

## 09.15.1 - Ingress Resource
Un _Ingress Resource_ es un conjunto de reglas y configuraciones aplicadas en el _Ingress Controller_.

Puede configurar reglas simplemente para reenviar todo el tráfico entrante a una sola aplicación o enrutar el tráfico a difrentes apilicaciones basado en URLs.

Se crea mediante un archivo de definición de kubernetes:
```yaml
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: ingress-wear
spec:
  backend:
      serviceName: wear-service
      servicePort: 80
```

- Ingress para diferentes paths
```yaml
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: ingress-wear-watch
spec:
  rules:
    - http:
        paths:
          - path: /wear
            backend:
			  serviceName: wear-service
			  servicePort: 80
		  - path: /watch
            backend:
			  serviceName: watch-service
			  servicePort: 80
```

- Otro tipo de configuración, partiendo de dos dominios:
```yaml
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: ingress-wear-watch
spec:
  rules:
	- host: wear.my-online-store.com
	  http:
        paths:
          - backend:
			  serviceName: wear-service
			  servicePort: 80
	- host: watch.my-online-store.com
	  http:
        paths:
          - backend:
			  serviceName: watch-service
			  servicePort: 80
```


## 09.15.2 - Ingress Annotations and rewrite-target
Los diferentes Ingress Controller tienen difrentes opciones que pueden ser utilizadas para personalizar su comportamiento. 
- Rewrite Target

Una web muestra su contenido en: `http://<watch-service>:<port>/`

Y la otra muestra su contenido en `http://<wear-service>:<port>/`

Para lograr lo anterior, necesitamos configurar el Ingress Controller para que podamos reenviar el tráfico al backendo correcto. Las aplicaciones no poseen ese path:

```bash
http://<ingress-service>:<ingress-port>/watch --> http://<watch-service>:<port>/

http://<ingress-service>:<ingress-port>/wear --> http://<wear-service>:<port>/

Without the rewrite-target option, this is what would happen:
```

Sin la opción rewrite-target, lo que pasaría sería lo siguiente:
```bash
http://<ingress-service>:<ingress-port>/watch --> http://<watch-service>:<port>/watch

http://<ingress-service>:<ingress-port>/wear --> http://<wear-service>:<port>/wear
```

Ejemplo de rewrite-target:

```yaml
    apiVersion: extensions/v1beta1
    kind: Ingress
    metadata:
      name: test-ingress
      namespace: critical-space
      annotations:
        nginx.ingress.kubernetes.io/rewrite-target: /
    spec:
      rules:
      - http:
          paths:
          - path: /pay
            backend:
              serviceName: pay-service
			  servicePort: 8282
```

In another example given here, this could also be:
`replace("/something(/|$)(.*)", "/$2")`
```yaml
    apiVersion: extensions/v1beta1
    kind: Ingress
    metadata:
      annotations:
        nginx.ingress.kubernetes.io/rewrite-target: /$2
      name: rewrite
      namespace: default
    spec:
      rules:
      - host: rewrite.bar.com
        http:
          paths:
          - backend:
              serviceName: http-svc
              servicePort: 80
            path: /something(/|$)(.*)
```


# 10 - Desing a Kubernetes Cluster
Algunas preguntas que debe realizar para poder diseñar un cluster.
- ¿Propósito?
  - Aprendizaje
  - Desarrollo y testeo
  - Producción
- Cloud o OnPrem?
- ¿Que cargas va a soportar el cluster?
  - ¿Cuantas aplicaciones se alojarán?
  - Tipo de aplicaciones
    - Aplicaciones web
    - Big Data / Analítica
  - Requisitos
    - CPU
    - Memoria
  - Tráfico
    - Continuo
    - Pesado

## 10.1 - Propósito
* Soluciones para fines educativos:
  * Minikube
  * Mono-nodo con kubeadm/GCP/AWS
  
* Soluciones para fines de desarrollo y testeo:
  * Multi-nodo con un Master y multiples workers
  * Setup usando kubeadm tool o provisión mediante GKE(Google Cloud), EKS(AWS) o AKS(Azure)

* Soluciones para fines productivos:
  * HA Multi-nodo con múltiples masters
  * kubeamd, GCP, Kops con AWS o plataformas soportadas
  * Hasta 5000 nodos
  * Hasta 150K PODs en el cluster
  * Hasta 300K contenedores
  * Hasta 100 PODs por nodo.

![machine type](../img/10_machine_type.png)

## 10.2 - Cloud o OnPrem
Para cloud hay multitud de herramientas para implementar un cluster. Pero para clusters _OnPrem_, _kubeadm_ es una herramienta muy util.

### 10.2.1 - Storage
Dependiendo de las cargas de trabajo configuradas, sus configuraciones de nodo y disco serán diferentes.
- High Perfomance - SSD Backed storage
- Multiple Concurrent connections - Network based storage
- Persistent shared volumes for shared access across multiple PODs
- Label nodes with specific disk types
- Use Node Selector to assing applications to nodes with specific disk types

### 10.2.2 - Nodes
- Virtual o physical Machines
- Minimum of 4 Node Cluster (size based on workload)
- Master vs Worker Nodes
- Linux X86_64 Architecture
  
### 10.2.3 - Master Nodes
En clusters pequeños es posible que almacene todos los componentes del plano de control en los masters, sin embargo en cluster grandes, es conveniente separar ETCD del nodo maestro

![master_node](../img/10_master_node.png)



# 10.3 - Configure HA
Si tuvieramos dos masters, cada API Server tendría una dirección distinta:
```bash
https://master1:6443

https://master2:6443
```

Desde el fichero kubeconfig, apuntabamos a un master, cuando existe más de uno utilizamos un Balanceador de carga por delante y este equilibra la carga.

Con respecto al __Scheduler__ o al __Controller Manager__, no se ejecutan en todos los Master Nodes activamente. Cuando se programa algún objecto, uno de ellos se vuelve pasivo, y se bloquea hasta para no aceptar la misma solicitud.
Para configurar el estado de estos servicios
```bash
kube-controller-manager --leader-elect true                 # Posibilidad de ser lider
                        --leader-elect-lease-duration 15s   # Si otro master, se vuelve en lider, se bloquea durante 15s
                        --leader-elect-renew-deadline 10s   # El proceso se renueva cada 10s
                        --leader-elect-retry-period 2s      # Tratan de convertirse en lider cada 2s
                        [other options]
```

Sobre ETCD, es interesante desacoplarlo de los masters, permitiendo así tener un menor riesgo a peder redudancia de datos si un nodo cae.


# 10.4 - ETCD en HA
ETCD es un sistema distribuido clave-valor, simple seguro y rápido.

* Distribuido
Es posible tener sus datos almacenamos en varias replicas en ETCD, todos mantienen una copia idéntica.
* Consistencia
ETCD asegura tener una copia consistenten en cada servidor. Puede leer de todas las instancias pero ETCD no procesa las escrituras en cada nodo, solo una de las intancias es responsable del procesamiento de las escrituras, de todos los nodos disponibles, uno es el lider, y el resto se convierten en seguidores. El lider asegura que todos los nodos posean los mismos datos.internamente se reenvian las operacionse de escritura al lider.

Para la elección del lider se implementa consenso distribuido utilizando el protocolo RAFT.

RAFT utiliza temporizadoresa aleatorios para iniciar solicitudes en cada nodo, el primero en terminar envia una solicitud al resto pidiendo ser el lider, el resto responden la solicitud, y el nodo principal asume su rol de Lider. En caso de caer uno de los nodos, los restantes vuelven a evaluar quien será lider.

En el caso de que un nodo caiga, cuando el nodo Lider, evalue si las copias enviadas al resto de nodos han sido enviadas correctamente, se evalua un número _MAJORITY_ (quorum), mínimo de nodos disponibles para que el cluster pueda funcionar. Por ello se recomienda tener al menos 3 nodos, o números impares para evitar posibles problemas con la segmentación de redes:
```bash
Quorum = N/2 + 1

# Examples
Quorum of 2 = 2/2 +1 = 2

Quorum of 3 = 3/2 +1 = 2.5 ~= 2
Quorum of 5 = 5/3 +1 = 3.5 ~= 3
```



# Install kubernetes hard way



# install kubernetes the kubeadm



# End to End test on a Kubernetes cluster



# 11 - Troubleshooting
## 11.1 - Fallos de aplicación
Partiendo de una aplicación web con dos componentes: Servidor Web y BBDD. Debemos comprender todos los componentes/objetos que existen en todo el flujo.

![application](../img/11_troubleshooting_applications.png)

1. Compruebe que su aplicación es accesible, por ejemplo con un curl.

```bash
curl http://web-service-ip:node-port
curl: (7) Failed to connect to web-service-ip port node-port: Connection timed out
```

2. Compruebe el servicio (Service aka SVC), fijese si contiene Endpoints asociados y los selectores del servicio y del pod.
```bash

kubectl describe service web-service

Name: web-service
Namespace: default
Labels: <none>
Annotations: <none>
Selector: name=webapp-mysql
Type: NodePort
IP: 10.96.0.156
Port: <unset> 8080/TCP
TargetPort: 8080/TCP
NodePort: <unset> 31672/TCP
Endpoints: 10.32.0.6:8080
Session Affinity: None
External Traffic Policy: Cluster
Events: <none>
```

```yaml
apiVersion: v1
kind: Pod
metadata:
name: webapp-mysql
labels:
app: example-app
name: webapp-mysql
spec:
containers:
- name: webapp-mysql
image: simple-webapp-mysql
ports:
- containerPort: 8080
```

3. Compruebe el estado del Pod, fijese en el número de reinicios, realice un describe al pod, y compruebe los logs

```bash
kubectl get pod
NAME     READY   STATUS   RESTARTS   AGE
Web      1/1     Running  5          50m
```

```bash
kubectl describe pod web
Events:
Type    Reason     Age   From               Message
----    ------     ----  ----               -------
Normal  Scheduled  52m   default-scheduler  Successfully assigned webapp-mysql to worker-1
Normal  Pulling    52m   kubelet, worker-1  pulling image "simple-webapp-mysql"
Normal  Pulled     52m   kubelet, worker-1  Successfully pulled image "simple-webapp-mysql"
Normal  Created    52m   kubelet, worker-1  Created container
Normal  Started    52m   kubelet, worker-1  Started container
```

```bash
kubectl logs web -f(same tail -f) --previous (show previous pod)
...
10.32.0.1 - - [01/Apr/2019 12:51:55] "GET / HTTP/1.1" 200 -
10.32.0.1 - - [01/Apr/2019 12:51:55] "GET /static/img/success.jpg HTTP/1.1" 200 -
10.32.0.1 - - [01/Apr/2019 12:51:55] "GET /favicon.ico HTTP/1.1" 404 -
10.32.0.1 - - [01/Apr/2019 12:51:57] "GET / HTTP/1.1" 200 -
10.32.0.1 - - [01/Apr/2019 12:51:57] "GET / HTTP/1.1" 200 -
10.32.0.1 - - [01/Apr/2019 12:51:58] "GET / HTTP/1.1" 200 -
10.32.0.1 - - [01/Apr/2019 12:51:58] "GET / HTTP/1.1" 200 –
10.32.0.1 - - [01/Apr/2019 12:51:55] "GET / HTTP/1.1" 200 -
10.32.0.1 - - [01/Apr/2019 12:51:55] "GET /static/img/success.jpg HTTP/1.1" 200 -
10.32.0.1 - - [01/Apr/2019 12:51:55] "GET /favicon.ico HTTP/1.1" 404 -
10.32.0.1 - - [01/Apr/2019 12:51:57] "GET / HTTP/1.1" 200 -
10.32.0.1 - - [01/Apr/2019 12:51:57] "GET / HTTP/1.1" 200 -
10.32.0.1 - - [01/Apr/2019 12:51:58] "GET / HTTP/1.1" 200 -
10.32.0.1 - - [01/Apr/2019 12:51:58] "GET / HTTP/1.1" 200 –
10.32.0.1 - - [01/Apr/2019 12:51:58] "GET / HTTP/1.1" 400 – Some Database Error application exiting!
```

4. Continuando con el análisis, compruebe el servicio de la BBDD y el Pod que contiene la BBDD.
5. Puede consultar la página de Kubernetes (https://kubernetes.io/docs/tasks/debug-application-cluster/debug-application/), aparcen algunos ejemplos más sobre troubleshooting de aplicaciones.

## 11.2 - Fallos en Control Plane (Plano de control)
En este punto veremos comprobar fallos en los componentes del Plano de Control (Control Plane).

1. Comprobaremos el estado de los nodos

```bash
kubectl get nodes

NAME     STATUS ROLES   AGE VERSION
worker-1 Ready  <none>  8d  v1.13.0
worker-2 Ready  <none>  8d  v1.13.0
```

2. Comprobaremos el estado de los pods del Cluster

```bash
kubectl get pods

NAME         READY STATUS  RESTARTS AGE
mysql        1/1   Running 0        113m
webapp-mysql 1/1   Running 0        113m
```

```bash
kubectl get pods -n kube-system

NAME                           READY STATUS  RESTARTS AGE
coredns-78fcdf6894-5dntv       1/1   Running 0        1h
coredns-78fcdf6894-knpzl       1/1   Running 0        1h
etcd-master                    1/1   Running 0        1h
kube-apiserver-master          1/1   Running 0        1h
kube-controller-manager-master 1/1   Running 0        1h
kube-proxy-fvbpj               1/1   Running 0        1h
kube-proxy-v5r2t               1/1   Running 0        1h
kube-scheduler-master          1/1   Running 0        1h
weave-net-7kd52                2/2   Running 1        1h
weave-net-jtl5m                2/2   Running 1        1h
```

3. Si tiene los componentes del Plano de Control como servicio, compruebe su estado

```bash
service kube-apiserver status

● kube-apiserver.service - Kubernetes API Server
Loaded: loaded (/etc/systemd/system/kube-apiserver.service; enabled; vendor preset: enabled)
Active: active (running) since Wed 2019-03-20 07:57:25 UTC; 1 weeks 1 days ago
Docs: https://github.com/kubernetes/kubernetes
Main PID: 15767 (kube-apiserver)
Tasks: 13 (limit: 2362)
```

```bash
service kube-controller-manager status

● kube-controller-manager.service - Kubernetes Controller Manager
Loaded: loaded (/etc/systemd/system/kube-controller-manager.service; enabled; vendor preset: enabled)
Active: active (running) since Wed 2019-03-20 07:57:25 UTC; 1 weeks 1 days ago
Docs: https://github.com/kubernetes/kubernetes
Main PID: 15771 (kube-controller)
Tasks: 10 (limit: 2362)
```

```bash
service kube-scheduler status

● kube-scheduler.service - Kubernetes Scheduler
Loaded: loaded (/etc/systemd/system/kube-scheduler.service; enabled; vendor preset: enabled)
Active: active (running) since Fri 2019-03-29 01:45:32 UTC; 11min ago
Docs: https://github.com/kubernetes/kubernetes
Main PID: 28390 (kube-scheduler)
Tasks: 10 (limit: 2362)
```

```bash
service kubelet status

● kubelet.service - Kubernetes Kubelet
Loaded: loaded (/etc/systemd/system/kubelet.service; enabled; vendor preset: enabled)
Active: active (running) since Wed 2019-03-20 14:22:06 UTC; 1 weeks 1 days ago
Docs: https://github.com/kubernetes/kubernetes
Main PID: 1281 (kubelet)
Tasks: 24 (limit: 1152)
```

```bash
service kube-proxy status

● kube-proxy.service - Kubernetes Kube Proxy
Loaded: loaded (/etc/systemd/system/kube-proxy.service; enabled; vendor preset: enabled)
Active: active (running) since Wed 2019-03-20 14:21:54 UTC; 1 weeks 1 days ago
Docs: https://github.com/kubernetes/kubernetes
Main PID: 794 (kube-proxy)
Tasks: 7 (limit: 1152)
```

4. Comprueba los logs de los distintos pods del Plano de control.

```bash
kubectl logs kube-apiserver-master -n kube-system

I0401 13:45:38.190735 1 server.go:703] external host was not specified, using 172.17.0.117
I0401 13:45:38.194290 1 server.go:145] Version: v1.11.3
I0401 13:45:38.819705 1 plugins.go:158] Loaded 8 mutating admission controller(s) successfully in the following order:
NamespaceLifecycle,LimitRanger,ServiceAccount,NodeRestriction,Priority,DefaultTolerationSeconds,DefaultStorageClass,MutatingAdmissionWebhook.
I0401 13:45:38.819741 1 plugins.go:161] Loaded 6 validating admission controller(s) successfully in the following order:
LimitRanger,ServiceAccount,Priority,PersistentVolumeClaimResize,ValidatingAdmissionWebhook,ResourceQuota.
I0401 13:45:38.821372 1 plugins.go:158] Loaded 8 mutating admission controller(s) successfully in the following order:
NamespaceLifecycle,LimitRanger,ServiceAccount,NodeRestriction,Priority,DefaultTolerationSeconds,DefaultStorageClass,MutatingAdmissionWebhook.
I0401 13:45:38.821410 1 plugins.go:161] Loaded 6 validating admission controller(s) successfully in the following order:
LimitRanger,ServiceAccount,Priority,PersistentVolumeClaimResize,ValidatingAdmissionWebhook,ResourceQuota.
I0401 13:45:38.985453 1 master.go:234] Using reconciler: lease
W0401 13:45:40.900380 1 genericapiserver.go:319] Skipping API batch/v2alpha1 because it has no resources.
W0401 13:45:41.370677 1 genericapiserver.go:319] Skipping API rbac.authorization.k8s.io/v1alpha1 because it has no resources.
W0401 13:45:41.381736 1 genericapiserver.go:319] Skipping API scheduling.k8s.io/v1alpha1 because it has no resources.
```

```bash
sudo journalctl -u kube-apiserver

Mar 20 07:57:25 master-1 systemd[1]: Started Kubernetes API Server.
Mar 20 07:57:25 master-1 kube-apiserver[15767]: I0320 07:57:25.553377 15767 flags.go:33] FLAG: --address="127.0.0.1"
Mar 20 07:57:25 master-1 kube-apiserver[15767]: I0320 07:57:25.558273 15767 flags.go:33] FLAG: --admission-control="[]"
Mar 20 07:57:25 master-1 kube-apiserver[15767]: I0320 07:57:25.558325 15767 flags.go:33] FLAG: --admission-control-config-file=""
Mar 20 07:57:25 master-1 kube-apiserver[15767]: I0320 07:57:25.558339 15767 flags.go:33] FLAG: --advertise-address="192.168.5.11"
Mar 20 07:57:25 master-1 kube-apiserver[15767]: I0320 07:57:25.558353 15767 flags.go:33] FLAG: --allow-privileged="true"
Mar 20 07:57:25 master-1 kube-apiserver[15767]: I0320 07:57:25.558365 15767 flags.go:33] FLAG: --alsologtostderr="false"
Mar 20 07:57:25 master-1 kube-apiserver[15767]: I0320 07:57:25.558413 15767 flags.go:33] FLAG: --anonymous-auth="true"
Mar 20 07:57:25 master-1 kube-apiserver[15767]: I0320 07:57:25.558425 15767 flags.go:33] FLAG: --api-audiences="[]"
Mar 20 07:57:25 master-1 kube-apiserver[15767]: I0320 07:57:25.558442 15767 flags.go:33] FLAG: --apiserver-count="3"
Mar 20 07:57:25 master-1 kube-apiserver[15767]: I0320 07:57:25.558454 15767 flags.go:33] FLAG: --audit-dynamic-configuration="false"
Mar 20 07:57:25 master-1 kube-apiserver[15767]: I0320 07:57:25.558464 15767 flags.go:33] FLAG: --audit-log-batch-buffer-size="10000"
Mar 20 07:57:25 master-1 kube-apiserver[15767]: I0320 07:57:25.558474 15767 flags.go:33] FLAG: --audit-log-batch-max-size="1"
Mar 20 07:57:25 master-1 kube-apiserver[15767]: I0320 07:57:25.558484 15767 flags.go:33] FLAG: --audit-log-batch-max-wait="0s"
Mar 20 07:57:25 master-1 kube-apiserver[15767]: I0320 07:57:25.558495 15767 flags.go:33] FLAG: --audit-log-batch-throttle-burst="0"
Mar 20 07:57:25 master-1 kube-apiserver[15767]: I0320 07:57:25.558504 15767 flags.go:33] FLAG: --audit-log-batch-throttle-enable="false"
Mar 20 07:57:25 master-1 kube-apiserver[15767]: I0320 07:57:25.558514 15767 flags.go:33] FLAG: --audit-log-batch-throttle-qps="0"
Mar 20 07:57:25 master-1 kube-apiserver[15767]: I0320 07:57:25.558528 15767 flags.go:33] FLAG: --audit-log-format="json"
```

## 11.3 - Fallos en nodos Worker
Cuando experimentamos fallos en los Nodos Worker, es un buen comienzo, realizar el siguiente análisis.

1. Comprobamos el estado de los nodos
```bash
kubectl get nodes

NAME     STATUS   ROLES  AGE VERSION
worker-1 Ready    <none> 8d  v1.13.0
worker-2 NotReady <none> 8d  v1.13.0
```

2. Comprobamos el estado de nodo `NotReady` 
Cada nodo tiene una serie de condiciones, en la que pueden indicarnos el tipo de fallo, dependiendo del estado, se establecen `True`, `False` o `Unknown`.

> Cuando el nodo no tiene espacio `OutOfDisk` es `True`
> Cuando el nodo no tiene memoria `MemoryPressure` es `True`
> Cuando el nodo tiene poca capacidad `DiskPressure` es `True`
> Cuando en el nodo hay demasiados procesos `PIDPressure` es `True`
> Si el nodo en conjunto es saludable el indicador `Ready` se establece en `True`

```bash
kubectl describe node worker-1

...
Conditions:
Type           Status   LastHeartbeatTime                Reason                      Message
----           ------   -----------------                ------                      -------
OutOfDisk      False    Mon, 01 Apr 2019 14:30:33 +0000  KubeletHasSufficientDisk    kubelet has sufficient disk space available
MemoryPressure False    Mon, 01 Apr 2019 14:30:33 +0000  KubeletHasSufficientMemory  kubelet has sufficient memory available
DiskPressure   False    Mon, 01 Apr 2019 14:30:33 +0000  KubeletHasNoDiskPressure    kubelet has no disk pressure
PIDPressure    False    Mon, 01 Apr 2019 14:30:33 +0000  KubeletHasSufficientPID     kubelet has sufficient PID available
Ready          True     Mon, 01 Apr 2019 14:30:33 +0000  KubeletReady                kubelet is posting ready status. AppArmor enabled
```

Cuando los estados se muestran `Unknown`, puede deberse a una pérdida del nodo, o puede que esté bloqueado.
```bash
kubectl describe node worker-1

...
Conditions: 
Type           Status   LastHeartbeatTime                Reason                   Message
----           ------   -----------------                ------                   -------
OutOfDisk      Unknown  Mon, 01 Apr 2019 14:20:20 +0000  NodeStatusUnknown        Kubelet stopped posting node status.
MemoryPressure Unknown  Mon, 01 Apr 2019 14:20:20 +0000  NodeStatusUnknown        Kubelet stopped posting node status.
DiskPressure   Unknown  Mon, 01 Apr 2019 14:20:20 +0000  NodeStatusUnknown        Kubelet stopped posting node status.
PIDPressure    False    Mon, 01 Apr 2019 14:20:20 +0000  KubeletHasSufficientPID  kubelet has sufficient PID available
Ready          Unknown  Mon, 01 Apr 2019 14:20:20 +0000  NodeStatusUnknown        Kubelet stopped posting node status.
```

3. Compruebe el estado del nodo entrando en el, con comandos como `top`, `df -h`, `free -mta`

4. Compruebe el estado de kubelet
```bash
service kubelet status

● kubelet.service - Kubernetes Kubelet
Loaded: loaded (/etc/systemd/system/kubelet.service; enabled; vendor preset: enabled)
Active: active (running) since Wed 2019-03-20 14:22:06 UTC; 1 weeks 1 days ago
Docs: https://github.com/kubernetes/kubernetes
Main PID: 1281 (kubelet)
Tasks: 24 (limit: 1152)
```

```bash
sudo journalctl –u kubelet

-- Logs begin at Wed 2019-03-20 05:30:37 UTC, end at Mon 2019-04-01 14:42:42 UTC. --
Mar 20 08:12:59 worker-1 systemd[1]: Started Kubernetes Kubelet.
Mar 20 08:12:59 worker-1 kubelet[18962]: Flag --tls-cert-file has been deprecated, This parameter should be set via the config file specified by
the Kubele
Mar 20 08:12:59 worker-1 kubelet[18962]: Flag --tls-private-key-file has been deprecated, This parameter should be set via the config file
specified by the
Mar 20 08:12:59 worker-1 kubelet[18962]: I0320 08:12:59.915179 18962 flags.go:33] FLAG: --address="0.0.0.0"
Mar 20 08:12:59 worker-1 kubelet[18962]: I0320 08:12:59.918149 18962 flags.go:33] FLAG: --allow-privileged="true"
Mar 20 08:12:59 worker-1 kubelet[18962]: I0320 08:12:59.918339 18962 flags.go:33] FLAG: --allowed-unsafe-sysctls="[]"
Mar 20 08:12:59 worker-1 kubelet[18962]: I0320 08:12:59.918502 18962 flags.go:33] FLAG: --alsologtostderr="false"
Mar 20 08:12:59 worker-1 kubelet[18962]: I0320 08:12:59.918648 18962 flags.go:33] FLAG: --anonymous-auth="true"
Mar 20 08:12:59 worker-1 kubelet[18962]: I0320 08:12:59.918841 18962 flags.go:33] FLAG: --application-metrics-count-limit="100"
Mar 20 08:12:59 worker-1 kubelet[18962]: I0320 08:12:59.918974 18962 flags.go:33] FLAG: --authentication-token-webhook="false"
Mar 20 08:12:59 worker-1 kubelet[18962]: I0320 08:12:59.919096 18962 flags.go:33] FLAG: --authentication-token-webhook-cache-ttl="2m0s"
Mar 20 08:12:59 worker-1 kubelet[18962]: I0320 08:12:59.919299 18962 flags.go:33] FLAG: --authorization-mode="AlwaysAllow"
Mar 20 08:12:59 worker-1 kubelet[18962]: I0320 08:12:59.919466 18962 flags.go:33] FLAG: --authorization-webhook-cache-authorized-ttl="5m0s"
Mar 20 08:12:59 worker-1 kubelet[18962]: I0320 08:12:59.919598 18962 flags.go:33] FLAG: --authorization-webhook-cache-unauthorized-ttl="30s"
Mar 20 08:12:59 worker-1 kubelet[18962]: I0320 08:12:59.919791 18962 flags.go:33] FLAG: --azure-container-registry-config=""
Mar 20 08:12:59 worker-1 kubelet[18962]: I0320 08:12:59.919971 18962 flags.go:33] FLAG: --boot-id-file="/proc/sys/kernel/random/boot_id"
Mar 20 08:12:59 worker-1 kubelet[18962]: I0320 08:12:59.920102 18962 flags.go:33] FLAG: --bootstrap-checkpoint-path=""
```

5. Compruebes los certificados de kubelete, caducidad, CA correcta.

```bash
openssl x509 -in /var/lib/kubelet/worker-1.crt -text
Certificate:
Data:
Version: 3 (0x2)
Serial Number:
ff:e0:23:9d:fc:78:03:35
Signature Algorithm: sha256WithRSAEncryption
Issuer: CN = KUBERNETES-CA
Validity
Not Before: Mar 20 08:09:29 2021 GMT
Not After : Apr 29 08:09:29 2021 GMT
Subject: CN = system:node:worker-1, O = system:nodes
Subject Public Key Info:
Public Key Algorithm: rsaEncryption
Public-Key: (2048 bit)
Modulus:
00:b4:28:0c:60:71:41:06:14:46:d9:97:58:2d:fe:
a9:c7:6d:51:cd:1c:98:b9:5e:e6:e4:02:d3:e3:71:
58:a1:60:fe:cb:e7:9b:4b:86:04:67:b5:4f:da:d6:
6c:08:3f:57:e9:70:59:57:48:6a:ce:e5:d4:f3:6e:
b2:fa:8a:18:7e:21:60:35:8f:44:f7:a9:39:57:16:
4f:4e:1e:b1:a3:77:32:c2:ef:d1:38:b4:82:20:8f:
11:0e:79:c4:d1:9b:f6:82:c4:08:84:84:68:d5:c3:
e2:15:a0:ce:23:3c:8d:9c:b8:dd:fc:3a:cd:42:ae:
5e:1b:80:2d:1b:e5:5d:1b:c1:fb:be:a3:9e:82:ff:
a1:27:c8:b6:0f:3c:cb:11:f9:1a:9b:d2:39:92:0e:
47:45:b8:8f:98:13:c6:4d:6a:18:75:a4:01:6f:73:
f6:f8:7f:eb:5d:59:94:46:d8:da:37:75:cf:27:0b:
39:7f:48:20:c5:fd:c7:a7:ce:22:9a:33:4a:30:1d:
95:ef:00:bd:fe:47:22:42:44:99:77:5a:c4:97:bb:
37:93:7c:33:64:f4:b8:3a:53:8c:f4:10:db:7f:5f:
2b:89:18:d6:0e:68:51:34:29:b1:f1:61:6b:4b:c6:
...
```

## 11.4 -Troubleshooting de red
Kubernetes utiliza plugins CNI para configurar la red. El kubelet se encarga de ejecutar los plugins.

* `cni-bin-dir`: Kubelet busca este directorio en busca de plugins en el arranque
* `network-plugin`: El plugin de red a utilizar desde cni-bin-dir. Debe coincidir con el nombre reportado por un plugin del directorio de plugins.

Hay varios plugins disponibles y estos son algunos:

* Weave: Es el único plugin mencionado en la documentación de Kubernetes.
* Flannel: Por ahora no soporta las `Network Policies` de Kubernetes.
* Calico: Es el CNI más maduro.

> Nota: Si hay varios archivos de configuración CNI en el directorio,  kubelet utilizará el archivo de configuración que viene primero por nombre en orden lexicográfico.

### 11.4.1 - DNS en Kubernetes

Kubernetes utiliza `CoreDNS`, es un servidor DNS flexible y extensible que puede servir como DNS del clúster Kubernetes.

> En clusters Kubernetes a gran escala, el uso de memoria de CoreDNS se ve afectado predominantemente por el número de Pods y Servicios en el cluster. Otros factores incluyen el tamaño de la caché de respuestas DNS llena, y la tasa de consultas recibidas (QPS) por instancia CoreDNS.

Los recursos de Kubernetes para coreDNS son:
* Service Account, coredns
* ClusterRoles, coredns y kube-dns
* ClusterRoleBindings, coredns y kube-dns
* Deployment, coredns
* Configmap, coredns
* Service, kube-dns

Al analizar el despliegue de coreDNS se puede ver que el `plugin Corefile` consiste en una configuración importante que se define como un __configmap__.

El puerto 53 es el usado para la resolución DNS.

```json
kubernetes cluster.local in-addr.arpa ip6.arpa {
    pods insecure
    fallthrough in-addr.arpa ip6.arpa
    ttl 30
}
```

Este es el backend de k8s para cluster.local y dominios inversos.

`proxy . /etc/resolv.conf`

Reenvía los dominios fuera del clúster directamente al servidor DNS autoritativo correcto.


### 11.4.2 - Troubleshooting para coreDNS

1. Si encuentra pods `CoreDNS` en estado `Pending`, compruebe primero que el plugin de red está instalado.

2. Los Pods de `CoreDNS` tienen el estado `CrashLoopBackOff` o `Error`:

Si tiene nodos que ejecutan SELinux con una versión antigua de Docker puede experimentar un escenario en el que los pods de coredns no se inician. Para solucionarlo puedes probar una de las siguientes opciones:

* Actualizar a una versión más reciente de Docker.
* Desactivar SELinux.
* Modificar el despliegue de CoreDNSara establecer `allowPrivilegeEscalation` a `True`:

```bash
kubectl -n kube-system get deployment coredns -o yaml | \
  sed 's/allowPrivilegeEscalation: false/allowPrivilegeEscalation: true/g' | | \
  kubectl apply -f -
```

* Otra causa para que CoreDNS tenga `CrashLoopBackOff` es cuando un Pod CoreDNS desplegado en Kubernetes detecta un bucle.
Hay muchas maneras de solucionar este problema, algunas de ellas se enumeran aquí:

    * Añadir lo siguiente a su kubelet config yaml: __resolvConf: path-to-your-real-resolv-conf-file__ 
    Esta flag le dice a kubelet que pase un resolv.conf alternativo a Pods. Para los sistemas que utilizan __systemd-resolved__, `/run/systemd/resolve/resolv.conf` suele ser la ubicación del resolv.conf "real", aunque puede ser diferente dependiendo de su distribución.
    
    * Desactive la caché local de DNS en los nodos anfitriones, y restaure `/etc/resolv.conf `al original.
    
    * Una solución rápida es editar su __Corefile__, sustituyendo `forward . /etc/resolv.conf` por la dirección IP de su DNS de origen, por ejemplo `forward . 8.8.8.8`. 
    Pero esto sólo arregla el problema para CoreDNS, kubelet continuará reenviando el resolv.conf inválido a todos los Pods dnsPolicy por defecto, dejándolos incapaces de resolver DNS.

3. Si los pods CoreDNS y el servicio kube-dns están funcionando bien, compruebe que el servicio kube-dns tiene endpoints válidos.
```bash
kubectl -n kube-system get ep kube-dns
```
Si no hay `Endpoints` en el `Service`, inspeccionle y compruebe que utiliza `Selectors` y puertos correctos.

4. Compruebe la conectividad desde un pod al Service de CoreDNS.
5. Compruebe la resolución DNS del cluster desde un pod a un servicio.
```bash
nslookup <service-name>.<namespace>
```

### 11.4.3 - Troubleshooting para kube-proxy

`kube-proxy` es un proxy de red que se ejecuta en cada nodo del cluster y mantiene reglas de red en los nodos. 
Estas reglas de red permiten la comunicación de red a los Pods desde sesiones de red dentro o fuera del cluster.

> En un cluster configurado con kubeadm, puede encontrar kube-proxy como un daemonset.

kubeproxy se encarga de vigilar los servicios y el endpoint asociado a cada servicio. 
Cuando el cliente se va a conectar al servicio utilizando la IP virtual, el kubeproxy se encarga de enviar el tráfico a los pods reales.

Puede ver que el binario `kube-proxy` se ejecuta con el siguiente comando dentro del contenedor kube-proxy.

```bash
kubectl describe ds kube-proxy -n kube-system

...
    Command:
      /usr/local/bin/kube-proxy
      --config=/var/lib/kube-proxy/config.conf
      --hostname-override=$(NODE_NAME)
...
```

La configuración se obtiene del fichero `/var/lib/kube-proxy/config.conf` y podemos sustituir el nombre de host con el nombre del nodo en el que se está ejecutando el pod.
En el archivo de configuración definimos el `clusterCIDR`, el modo `kubeproxy`, `ipvs`, `iptables`, `bindaddress`, `kube-config`, etc.

Solución de problemas relacionados con kube-proxy
1. Compruebe que el Pod `kube-proxy` en el namespace `kube-system` se encuentra `Running`.
2. Compruebe los logs de `kube-proxy`.
3. Compruebe que el `Configmap` está correctamente definido y que el archivo de configuración para el binario kube-proxy en ejecución es correcto.
4. `kube-config `está definido en el `Confimap`.
5. Compruebe que `kube-proxy` se está ejecutando dentro del contenedor.

```bash
# netstat -plan | grep kube-proxy
tcp        0      0 0.0.0.0:30081           0.0.0.0:*               LISTEN      1/kube-proxy
tcp        0      0 127.0.0.1:10249         0.0.0.0:*               LISTEN      1/kube-proxy
tcp        0      0 172.17.0.12:33706       172.17.0.12:6443        ESTABLISHED 1/kube-proxy
tcp6       0      0 :::10256                :::*  
```



# 12 - Other Topics
## 12.1 - JSON PATH
## 12.2 - Advance Kubectl Commands



