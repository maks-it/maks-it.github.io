[Harbor](https://bitnami.com/stack/harbor/helm)
[Harbor Chart Docs](https://github.com/bitnami/charts/tree/main/bitnami/harbor/#installing-the-chart)

```bash
kubectl create namespace harbor-system
```

Create harbor-values.yml:

```yaml
volumePermissions:
  enabled: true

postgresql:
  auth:
    postgresPassword: password
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
  name: data-my-release-harbor-trivy-0
spec:
  capacity:
    storage: 5Gi
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  claimRef:
    namespace: harbor-system
    name: data-my-release-harbor-trivy-0
  hostPath:
    path: "/storage/pool-1/harbor/trivy"

---

apiVersion: v1
kind: PersistentVolume
metadata:
  name: data-my-release-postgresql-0
spec:
  capacity:
    storage: 8Gi
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  claimRef:
    namespace: harbor-system
    name: data-my-release-postgresql-0
  hostPath:
    path: "/storage/pool-1/harbor/postgresql"

---

apiVersion: v1
kind: PersistentVolume
metadata:
  name: my-release-harbor-jobservice
spec:
  capacity:
    storage: 1Gi
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  claimRef:
    namespace: harbor-system
    name: my-release-harbor-jobservice
  hostPath:
    path: "/storage/pool-1/harbor/jobservice"
---

apiVersion: v1
kind: PersistentVolume
metadata:
  name: my-release-harbor-registry
spec:
  capacity:
    storage: 5Gi
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  claimRef:
    namespace: harbor-system
    name: my-release-harbor-registry
  hostPath:
    path: "/storage/pool-1/harbor/registry"

---

apiVersion: v1
kind: PersistentVolume
metadata:
  name: redis-data-my-release-redis-master-0
spec:
  capacity:
    storage: 8Gi
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  claimRef:
    namespace: harbor-system
    name: redis-data-my-release-redis-master-0
  hostPath:
    path: "/storage/pool-1/harbor/redis"
     
```

For some reason registry db is not autocreated:

```bash
kubectl exec -it svc/my-release-postgresql -- /bin/bash
```

```bash
psql -U postgres
```

```bash
CREATE DATABASE registry;
```

```bash
\q
exit
```

## Port forwarding

```bash
kubectl port-forward --namespace harbor-system svc/my-release-harbor 443:8080
```

## Uninstall chart

```bash
helm delete my-release --namespace harbor-system 
```