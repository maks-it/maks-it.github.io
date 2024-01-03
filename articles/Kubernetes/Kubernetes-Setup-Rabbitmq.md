# Kubernetes Setup RabbitMQ

[RabbitMQ](https://bitnami.com/stack/rabbitmq/helm)
[RabbitMQ Chart Docs](https://github.com/bitnami/charts/tree/main/bitnami/rabbitmq/#installing-the-chart)

Create rabbitmq-system namespace on your kubernetes cluster

```bash
kubectl create namespace rabbitmq-system
```

Create storage class

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: rabbitmq-storage-class
  namespace: rabbitmq-system  # Assigning StorageClass to rabbitmq-system namespace
provisioner: kubernetes.io/no-provisioner
volumeBindingMode: WaitForFirstConsumer
```

Create values file:

```yaml
global:
  storageClass: "rabbitmq-storage-class"
persistence:
  size: 1Gi
volumePermissions:
  enabled: true
podManagementPolicy: Parallel
replicaCount: 3
clustering:
  forceBoot: true
auth:
  username: admin
  password: password
  erlangCookie: 1lOBkQT8vBdA4hfoGM8MkgzfEjA27chE
extraEnvVars:
  - name: LOG_LEVEL
    value: error
image:
  debug: true
```

> As the image runs as non-root by default, it is necessary to adjust the ownership of the persistent volume so that the container can write data into it.
>
>By default, the chart is configured to use Kubernetes Security Context to automatically change the ownership of the volume. However, this feature does not work in all Kubernetes distributions. As an alternative, this chart supports using an initContainer to change the ownership of the volume before mounting it in the final destination.
>
>You can enable this initContainer by setting volumePermissions.enabled to true.

```bash
helm install my-release oci://registry-1.docker.io/bitnamicharts/rabbitmq \
  --namespace rabbitmq-system \
  --values rabbitmq-values.yml
```

Check persistance volumes to be created

```bash
kubectl get pvc -n rabbitmq-system
```

```bash
NAME                         STATUS    VOLUME   CAPACITY   ACCESS MODES   STORAGECLASS             VOLUMEATTRIBUTESCLASS   AGE
data-my-release-rabbitmq-0   Pending                                      rabbitmq-storage-class   <unset>
```

Then create necessary folders on nfs server, add following to your storage yaml

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: data-my-release-rabbitmq-0-pv  # Matching the PVC name
spec:
  capacity:
    storage: 1Gi
  volumeMode: Filesystem
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain  # This can be adjusted based on your retention policy
  storageClassName: rabbitmq-storage-class
  hostPath:
    path: /storage/pool-1/rabbitmq  # Path on the node where the local storage is mounted
    type: DirectoryOrCreate  # You can use DirectoryOrCreate or Directory

---

apiVersion: v1
kind: PersistentVolume
metadata:
  name: data-my-release-rabbitmq-1-pv  # Matching the PVC name
spec:
  capacity:
    storage: 1Gi
  volumeMode: Filesystem
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain  # This can be adjusted based on your retention policy
  storageClassName: rabbitmq-storage-class
  hostPath:
    path: /storage/pool-1/rabbitmq  # Path on the node where the local storage is mounted
    type: DirectoryOrCreate  # You can use DirectoryOrCreate or Directory

---

apiVersion: v1
kind: PersistentVolume
metadata:
  name: data-my-release-rabbitmq-2-pv  # Matching the PVC name
spec:
  capacity:
    storage: 1Gi
  volumeMode: Filesystem
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain  # This can be adjusted based on your retention policy
  storageClassName: rabbitmq-storage-class
  hostPath:
    path: /storage/pool-1/rabbitmq  # Path on the node where the local storage is mounted
    type: DirectoryOrCreate  # You can use DirectoryOrCreate or Directory
```

## Port forward

To access for outside the cluster, perform the following steps:

To Access the RabbitMQ AMQP port:

```bash
    echo "URL : amqp://127.0.0.1:5672/"
    kubectl port-forward --namespace rabbitmq-system svc/my-release-rabbitmq 5672:5672
```
To Access the RabbitMQ Management interface:

```bash
    echo "URL : http://127.0.0.1:15672/"
    kubectl port-forward --namespace rabbitmq-system svc/my-release-rabbitmq 15672:15672
```

## Uninstall chart

```bash
helm delete my-release --namespace rabbitmq-system
```