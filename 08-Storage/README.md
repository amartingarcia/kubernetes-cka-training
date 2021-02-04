# 08 Storage
## 08.1 Volumes
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


## 08.2 Persistent Volumes
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


## 08.3 Persistent Volume Claims
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


## 08.4 PVCs in PODs
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



## 08.7 Storage Class
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