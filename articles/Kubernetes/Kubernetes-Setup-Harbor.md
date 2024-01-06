[Harbor](https://bitnami.com/stack/harbor/helm)
[Harbor Chart Docs](https://github.com/bitnami/charts/tree/main/bitnami/harbor/#installing-the-chart)

```bash
kubectl create namespace harbor-system
```

Create storage class:

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: harbor-storage-class
  namespace: harbor-system  # Assigning StorageClass to harbor-system namespace
provisioner: kubernetes.io/no-provisioner
volumeBindingMode: WaitForFirstConsumer
```

```yaml

```


```bash


helm install my-release oci://registry-1.docker.io/bitnamicharts/harbor  \
  --namespace harbor-system \
  --values harbor-values.yml

```

```bash
echo Username: "admin"
echo Password: $(kubectl get secret --namespace harbor-system my-release-harbor-core-envvars -o jsonpath="{.data.HARBOR_ADMIN_PASSWORD}" | base64 -d)
```


```bash
kubectl get pvc -n harbor-system
NAME                                   STATUS    VOLUME   CAPACITY   ACCESS MODES   STORAGECLASS           VOLUMEATTRIBUTESCLASS   AGE
data-my-release-harbor-trivy-0         Pending                                      harbor-storage-class   <unset>                 101s
data-my-release-postgresql-0           Pending                                      harbor-storage-class   <unset>                 101s
my-release-harbor-jobservice           Pending                                      harbor-storage-class   <unset>                 102s
my-release-harbor-registry             Pending                                      harbor-storage-class   <unset>                 102s
redis-data-my-release-redis-master-0   Pending                                      harbor-storage-class   <unset>                 101s
```

kubectl get pvc my-release-harbor-registry -o jsonpath='{.spec.resources.requests.storage}' -n harbor-system


kubectl exec -it my-release-harbor-core-7d5885d754-drzvm -n harbor-system -- /bin/sh
kubectl run -i --tty --rm debug --namespace=harbor-system --image=busybox --restart=Never -- ping my-release-redis-master

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: harbor-pv
spec:
  capacity:
    storage: 100Gi
  accessModes:
    - ReadWriteMany
  persistentVolumeReclaimPolicy: Retain
  storageClassName: harbor-storage-class
  hostPath:
    path: /storage/pool-1/harbor  # Path on the node where the local storage is mounted
    type: DirectoryOrCreate  # You can use DirectoryOrCreate or Directory
```




```bash
kubectl port-forward --namespace harbor-system svc/my-release-harbor 80:80
```

## Uninstall chart

```bash
kubectl delete pvc --all -n harbor-system
helm delete my-release --namespace harbor-system 
```