# Kubernetes Setup Redis

[Redis](https://bitnami.com/stack/redis/helm)
[Redis Chart Docs](https://github.com/bitnami/charts/tree/main/bitnami/redis/#installing-the-chart)

Create redis-system namespace on your kubernetes cluster

```bash
kubectl create namespace redis-system
```

Create storage class:

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: redis-storage-class
  namespace: redis-system  # Assigning StorageClass to redis-system namespace
provisioner: kubernetes.io/no-provisioner
volumeBindingMode: WaitForFirstConsumer
```

Create values file for replication:

```yaml
global:
  storageClass: redis-storage-class
persistence:
  size: 8Gi
volumePermissions:
  enabled: true
architecture: replication

replica:
  replicaCount: 3

networkPolicy:
  enabled: true

sentinel:
  enabled: true
  image:
    debug: true

auth:
  password: password

extraEnvVars:
  - name: LOG_LEVEL
    value: error
image:
  debug: true
```

Standalone:

```yaml
global:
  storageClass: redis-storage-class
persistence:
  size: 8Gi
volumePermissions:
  enabled: true
architecture: standalone
auth:
  password: password
extraEnvVars:
  - name: LOG_LEVEL
    value: error
image:
  debug: true
```

```bash
helm install my-release oci://registry-1.docker.io/bitnamicharts/redis \
  --namespace redis-system \
  --values redis-values.yml
```

Check persistance volumes to be created

```bash
kubectl get pvc -n redis-system
```

For replicaset:

```bash
NAME                                 STATUS    VOLUME   CAPACITY   ACCESS MODES   STORAGECLASS          VOLUMEATTRIBUTESCLASS   AGE
redis-data-my-release-redis-node-0   Pending                                      redis-storage-class   <unset>                 48s
redis-data-my-release-redis-replicas-0   Pending                                      redis-storage-class   <unset>                 2m44s
```

Then create necessary folders on nfs server, add following to your storage yaml

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: redis-data-my-release-redis-master-0-pv  # Matching the PVC name
spec:
  capacity:
    storage: 8Gi
  volumeMode: Filesystem
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain  # This can be adjusted based on your retention policy
  storageClassName: redis-storage-class
  hostPath:
    path: /storage/pool-1/redis  # Path on the node where the local storage is mounted
    type: DirectoryOrCreate  # You can use DirectoryOrCreate or Directory

---

apiVersion: v1
kind: PersistentVolume
metadata:
  name: redis-data-my-release-redis-node-0-pv  # Matching the PVC name
spec:
  capacity:
    storage: 8Gi
  volumeMode: Filesystem
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain  # This can be adjusted based on your retention policy
  storageClassName: redis-storage-class
  hostPath:
    path: /storage/pool-1/redis  # Path on the node where the local storage is mounted
    type: DirectoryOrCreate  # You can use DirectoryOrCreate or Directory
```

For standalone:

```bash
NAME                                     STATUS    VOLUME   CAPACITY   ACCESS MODES   STORAGECLASS          VOLUMEATTRIBUTESCLASS   AGE
redis-data-my-release-redis-master-0     Pending                                      redis-storage-class   <unset>                 2m44s
```

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: redis-data-my-release-redis-master-0-pv  # Matching the PVC name
spec:
  capacity:
    storage: 8Gi
  volumeMode: Filesystem
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain  # This can be adjusted based on your retention policy
  storageClassName: redis-storage-class
  hostPath:
    path: /storage/pool-1/redis  # Path on the node where the local storage is mounted
    type: DirectoryOrCreate  # You can use DirectoryOrCreate or Directory
```


## Scaling

```bash
kubectl scale statefulset my-release-rabbitmq --replicas=1 -n rabbitmq-system
```


## Port forwarding

```bash
 kubectl port-forward --namespace redis-system svc/my-release-redis 6379:6379 &
    REDISCLI_AUTH="$REDIS_PASSWORD" redis-cli -h 127.0.0.1 -p 6379
```

```bash
kubectl port-forward --namespace redis-system svc/my-release-redis-master 6379:6379 &
    REDISCLI_AUTH="$REDIS_PASSWORD" redis-cli -h 127.0.0.1 -p 6379
```

## Uninstall chart

```bash
helm delete my-release --namespace redis-system
```
