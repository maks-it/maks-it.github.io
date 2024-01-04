# Kubernetes Setup MongoDB

[MongoDB](https://bitnami.com/stack/mongodb/helm)
[MongoDB Chart Docs](https://github.com/bitnami/charts/tree/main/bitnami/mongodb/#installing-the-chart)

Create mongodb-system namespace on your kubernetes cluster

```bash
kubectl create namespace mongodb-system
```

Create storage class:

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: mongodb-storage-class
  namespace: mongodb-system  # Assigning StorageClass to mongodb-system namespace
provisioner: kubernetes.io/no-provisioner
volumeBindingMode: WaitForFirstConsumer
```

Create values file for replicaset:

```yaml
global:
  storageClass: mongodb-storage-class
persistence:
  size: 8Gi
volumePermissions:
  enabled: true
architecture: replicaset

podManagementPolicy: Parallel
replicaCount: 3
auth:
  rootUser: admin
  rootPassword: password
  replicaSetKey: replicasetkey
extraEnvVars:
  - name: LOG_LEVEL
    value: error
image:
  debug: true
```

For standalone:

```yaml
global:
  storageClass: mongodb-storage-class
persistence:
  size: 8Gi
volumePermissions:
  enabled: true
architecture: standalone
auth:
  rootUser: admin
  rootPassword: password
extraEnvVars:
  - name: LOG_LEVEL
    value: error
image:
  debug: true
```

```bash
helm install my-release oci://registry-1.docker.io/bitnamicharts/mongodb \
  --namespace mongodb-system \
  --values mongodb-values.yml
```

Check persistance volumes to be created

```bash
kubectl get pvc -n mongodb-system
```

In case of replicaset:

```bash
NAME                           STATUS    VOLUME   CAPACITY   ACCESS MODES   STORAGECLASS            VOLUMEATTRIBUTESCLASS   AGE
datadir-my-release-mongodb-0   Pending                                      mongodb-storage-class   <unset>                 28s
datadir-my-release-mongodb-1   Pending                                      mongodb-storage-class   <unset>                 18s
datadir-my-release-mongodb-2   Pending                                      mongodb-storage-class   <unset>                 18s
```

Then create necessary folders on nfs server, add following to your storage yaml


```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: datadir-my-release-mongodb-0-pv  # Matching the PVC name
spec:
  capacity:
    storage: 8Gi
  volumeMode: Filesystem
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain  # This can be adjusted based on your retention policy
  storageClassName: mongodb-storage-class
  hostPath:
    path: /storage/pool-1/mongodb  # Path on the node where the local storage is mounted
    type: DirectoryOrCreate  # You can use DirectoryOrCreate or Directory

---

apiVersion: v1
kind: PersistentVolume
metadata:
  name: datadir-my-release-mongodb-1-pv  # Matching the PVC name
spec:
  capacity:
    storage: 8Gi
  volumeMode: Filesystem
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain  # This can be adjusted based on your retention policy
  storageClassName: mongodb-storage-class
  hostPath:
    path: /storage/pool-1/mongodb  # Path on the node where the local storage is mounted
    type: DirectoryOrCreate  # You can use DirectoryOrCreate or Directory

---

apiVersion: v1
kind: PersistentVolume
metadata:
  name: datadir-my-release-mongodb-2-pv  # Matching the PVC name
spec:
  capacity:
    storage: 8Gi
  volumeMode: Filesystem
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain  # This can be adjusted based on your retention policy
  storageClassName: mongodb-storage-class
  hostPath:
    path: /storage/pool-1/mongodb  # Path on the node where the local storage is mounted
    type: DirectoryOrCreate  # You can use DirectoryOrCreate or Directory
```

In case of standalone:

```bash
NAME                           STATUS    VOLUME   CAPACITY   ACCESS MODES   STORAGECLASS            VOLUMEATTRIBUTESCLASS   AGE
my-release-mongodb             Pending                                                               mongodb-storage-class   <unset>                 22s
```

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: my-release-mongodb-pv  # Matching the PVC name
spec:
  capacity:
    storage: 8Gi
  volumeMode: Filesystem
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain  # This can be adjusted based on your retention policy
  storageClassName: mongodb-storage-class
  hostPath:
    path: /storage/pool-1/mongodb  # Path on the node where the local storage is mounted
    type: DirectoryOrCreate  # You can use DirectoryOrCreate or Directory
```

## Scaling (Replicaset only)

```bash
kubectl scale statefulset my-release-mongodb --replicas=1 -n mongodb-system
```

## Port forward

```bash
kubectl port-forward --namespace mongodb-system svc/my-release-mongodb 27017:27017 &
mongosh --host 127.0.0.1 --authenticationDatabase admin -p password
```

## Uninstall chart

```bash
helm delete my-release --namespace mongodb-system
```