# Setup Dapr to Kubernetes cluster

Make sure Helm 3 is installed on your machine

Add Helm repo and update

```bash
helm repo add dapr https://dapr.github.io/helm-charts/
helm repo update
```

Create dapr-system namespace on your kubernetes cluster

```bash
kubectl create namespace dapr-system
```

## Search for specific version inside repository

```bash
# See which chart versions are available
helm search repo dapr --devel --versions
```

```powershell
helm search repo dapr/dapr --versions | Select-Object -Skip 1 | ForEach-Object { ($_ -split '\s+')[1] } | Select-Object -First 1
```

## Install the Dapr chart on your cluster in the dapr-system namespace.

Create storage class:

[Production guidelines on Kubernetes](https://docs.dapr.io/operations/hosting/kubernetes/kubernetes-production/)

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: dapr-storage-class
  namespace: dapr-system  # Assigning StorageClass to dapr-system namespace
provisioner: kubernetes.io/no-provisioner
volumeBindingMode: WaitForFirstConsumer
```

Create `values.yml` with following content

```yml
global:
  ha:
    enabled: true
dapr_placement:
  volumeclaims:
    storageClassName: "dapr-storage-class"
    storageSize: 1Gi
```

For all availabe options consult [Helm chart readme](https://github.com/dapr/dapr/blob/master/charts/dapr/README.md)

```bash
helm upgrade --install dapr dapr/dapr \
  --version=1.11 \
  --namespace dapr-system \
  --values dapr-values.yml
```

## Verify installation

Once the chart installation is complete verify the dapr-operator, dapr-placement, dapr-sidecar-injector and dapr-sentry pods are running in the dapr-system namespace:

```bash
kubectl get pods -n dapr-system -w
```

```bash
NAME                                    READY   STATUS    RESTARTS      AGE
dapr-dashboard-5858b7d9d8-zjzdf         1/1     Running   0             11m
dapr-operator-56875b7cb8-jk9nq          1/1     Running   0             22m
dapr-operator-56875b7cb8-k26jb          1/1     Running   0             22m
dapr-operator-56875b7cb8-srgs2          1/1     Running   0             22m
dapr-placement-server-0                 0/1     Peding    0             22m
dapr-placement-server-1                 0/1     Peding    0             22m
dapr-placement-server-2                 0/1     Peding    0             22m
dapr-sentry-d6fc47c95-jq4vj             1/1     Running   0             22m
dapr-sentry-d6fc47c95-k5lr9             1/1     Running   0             22m
dapr-sentry-d6fc47c95-wxmlh             1/1     Running   0             22m
dapr-sidecar-injector-74d44dd96-ccmwl   1/1     Running   0             22m
dapr-sidecar-injector-74d44dd96-pf8rr   1/1     Running   0             22m
dapr-sidecar-injector-74d44dd96-vq8tv   1/1     Running   0             22m
```

## Dapr Persistance Volume missing problem

As you could already understand, something is not working as expected, there are 3 pods in Pendig status:

```bash
NAME                                    READY   STATUS    RESTARTS      AGE
dapr-dashboard-5858b7d9d8-zjzdf         1/1     Running   0             11m
dapr-operator-56875b7cb8-jk9nq          1/1     Running   0             22m
dapr-operator-56875b7cb8-k26jb          1/1     Running   0             22m
dapr-operator-56875b7cb8-srgs2          1/1     Running   0             22m
dapr-placement-server-0                 0/1     Peding    0             22m
dapr-placement-server-1                 0/1     Peding    0             22m
dapr-placement-server-2                 0/1     Peding    0             22m
dapr-sentry-d6fc47c95-jq4vj             1/1     Running   0             22m
dapr-sentry-d6fc47c95-k5lr9             1/1     Running   0             22m
dapr-sentry-d6fc47c95-wxmlh             1/1     Running   0             22m
dapr-sidecar-injector-74d44dd96-ccmwl   1/1     Running   0             22m
dapr-sidecar-injector-74d44dd96-pf8rr   1/1     Running   0             22m
dapr-sidecar-injector-74d44dd96-vq8tv   1/1     Running   0             22m
```

Let's try to understand what's going wrong:

```bash
kubectl get pvc -n dapr-system
```

```bash
NAME                               STATUS   VOLUME                                CAPACITY   ACCESS MODES   STORAGECLASS         VOLUMEATTRIBUTESCLASS   AGE
raft-log-dapr-placement-server-0   Bound    raft-log-dapr-placement-server-0-pv   1Gi        RWO            dapr-storage-class   <unset>                 31m
raft-log-dapr-placement-server-1   Bound    raft-log-dapr-placement-server-1-pv   1Gi        RWO            dapr-storage-class   <unset>                 31m
raft-log-dapr-placement-server-2   Bound    raft-log-dapr-placement-server-2-pv   1Gi        RWO            dapr-storage-class   <unset>                 31m
```

Let's see one of them with command and check what is wrong in Events:

```bash
kubectl describe pvc raft-log-dapr-placement-server-0 -n dapr-system
kubectl describe pvc raft-log-dapr-placement-server-1 -n dapr-system
kubectl describe pvc raft-log-dapr-placement-server-2 -n dapr-system
```

Then look ad pvc deployments:

```bash
kubectl get pvc raft-log-dapr-placement-server-0 -n dapr-system -o yaml
kubectl get pvc raft-log-dapr-placement-server-1 -n dapr-system -o yaml
kubectl get pvc raft-log-dapr-placement-server-2 -n dapr-system -o yaml
```

Now it's clear that this happens because dapr helm creates 3 `PersistentVolumeClaim`s and no `PersistentVolume` exists.
In few words `PersistentVolumeClaim` isn't able to find any sutable `PersistentVolume`... And I spend a lot of time to understand it...

So create 3 `PersistentVolume`s needed by `PersistentVolumeClaim`s generated by helm (Kubernetes will match them by similar name, wired...):

```yaml

apiVersion: v1
kind: PersistentVolume
metadata:
  name: raft-log-dapr-placement-server-0-pv  # Matching the PVC name
spec:
  capacity:
    storage: 1Gi
  volumeMode: Filesystem
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain  # This can be adjusted based on your retention policy
  storageClassName: dapr-storage-class
  hostPath:
    path: /storage/pool-1/dapr  # Path on the node where the local storage is mounted
    type: DirectoryOrCreate  # You can use DirectoryOrCreate or Directory

---

apiVersion: v1
kind: PersistentVolume
metadata:
  name: raft-log-dapr-placement-server-1-pv  # Matching the PVC name
spec:
  capacity:
    storage: 1Gi
  volumeMode: Filesystem
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain  # This can be adjusted based on your retention policy
  storageClassName: dapr-storage-class
  hostPath:
    path: /storage/pool-1/dapr  # Path on the node where the local storage is mounted
    type: DirectoryOrCreate  # You can use DirectoryOrCreate or Directory

---

apiVersion: v1
kind: PersistentVolume
metadata:
  name: raft-log-dapr-placement-server-2-pv  # Matching the PVC name
spec:
  capacity:
    storage: 1Gi
  volumeMode: Filesystem
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain  # This can be adjusted based on your retention policy
  storageClassName: dapr-storage-class
  hostPath:
    path: /storage/pool-1/dapr  # Path on the node where the local storage is mounted
    type: DirectoryOrCreate  # You can use DirectoryOrCreate or Directory

```

> Note: In this scenario `/storage/pool-1/dapr` is NFS volume mapped

```bash
kubectl get pv -n dapr-system
```

```bash
NAME                                  CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                                          STORAGECLASS         VOLUMEATTRIBUTESCLASS   REASON   AGE
raft-log-dapr-placement-server-0-pv   1Gi        RWO            Retain           Bound    dapr-system/raft-log-dapr-placement-server-0   dapr-storage-class   <unset>                          60m
raft-log-dapr-placement-server-1-pv   1Gi        RWO            Retain           Bound    dapr-system/raft-log-dapr-placement-server-1   dapr-storage-class   <unset>                          60m
raft-log-dapr-placement-server-2-pv   1Gi        RWO            Retain           Bound    dapr-system/raft-log-dapr-placement-server-2   dapr-storage-class   <unset>                          60m
```

All PVs are bound and pods are in ready state:

```bash
kubectl get pods -n dapr-system
```

```bash
NAME                                    READY   STATUS    RESTARTS      AGE
dapr-operator-56875b7cb8-jk9nq          1/1     Running   0             60m
dapr-operator-56875b7cb8-k26jb          1/1     Running   0             60m
dapr-operator-56875b7cb8-srgs2          1/1     Running   0             60m
dapr-placement-server-0                 1/1     Running   1 (60m ago)   60m
dapr-placement-server-1                 1/1     Running   0             60m
dapr-placement-server-2                 1/1     Running   0             60m
dapr-sentry-d6fc47c95-jq4vj             1/1     Running   0             60m
dapr-sentry-d6fc47c95-k5lr9             1/1     Running   0             60m
dapr-sentry-d6fc47c95-wxmlh             1/1     Running   0             60m
dapr-sidecar-injector-74d44dd96-ccmwl   1/1     Running   0             60m
dapr-sidecar-injector-74d44dd96-pf8rr   1/1     Running   0             60m
dapr-sidecar-injector-74d44dd96-vq8tv   1/1     Running   0             60m
```

## Install dapr dashboard

```bash
helm upgrade --install dapr-dashboard dapr/dapr-dashboard --version=0.14 --namespace dapr-system
```

## Install daprcli

Install from Terminal
Install the latest Linux Dapr CLI to /usr/local/bin:

```bash
wget -q https://raw.githubusercontent.com/dapr/cli/master/install/install.sh -O - | /bin/bash
```

## Overall installation status

```bash
dapr status -k
```

```bash
NAME                   NAMESPACE    HEALTHY  STATUS   REPLICAS  VERSION  AGE  CREATED
  dapr-placement-server  dapr-system  True     Running  3         1.11.6   43m  2023-12-25 01:47.15
  dapr-dashboard         dapr-system  True     Running  1         0.14.0   33m  2023-12-25 01:57.50
  dapr-operator          dapr-system  True     Running  3         1.11.6   43m  2023-12-25 01:47.15
  dapr-sentry            dapr-system  True     Running  3         1.11.6   43m  2023-12-25 01:47.15
  dapr-sidecar-injector  dapr-system  True     Running  3         1.11.6   43m  2023-12-25 01:47.15
```

## Port forwarding

```bash
kubectl port-forward svc/dapr-dashboard -n dapr-system 8081:8080
```

Now you able to open `http://localhost:8081` and start to use Dapr via its dashboard.

## Uninstall Dapr on Kubernetes

```bash
helm uninstall dapr -n dapr-system
helm uninstall dapr-dashboard -n dapr-system
```

## Appendix

### Dapr ports

You may check these ports by running this command:

```bash
kubectl get pods -n dapr-system -o=json | jq '.items[] | {name: .metadata.name, containerPorts: [.spec.containers[].ports[]? | {port: .containerPort, protocol: .protocol}]}'
```

|Protocol	|Direction	|Port Range	|Purpose	|Used By|
|--|--|--|--|--|
|TCP|Inbound|8080||Dashboard|


|Protocol	|Direction	|Port Range	|Purpose	|Used By|
|--|--|--|--|--|
|TCP|Inbound|6500||Operator|
|TCP|Inbound|9091||Operator|


|Protocol	|Direction	|Port Range	|Purpose	|Used By|
|--|--|--|--|--|
|TCP|Inbound|50005||Placement|
|TCP|Inbound|8201||Placement|
|TCP|Inbound|9091||Placement|


|Protocol	|Direction	|Port Range	|Purpose	|Used By|
|--|--|--|--|--|
|TCP|Inbound|50001||Sentry|
|TCP|Inbound|9091||Sentry|


|Protocol	|Direction	|Port Range	|Purpose	|Used By|
|--|--|--|--|--|
|TCP|Inbound|4000||Sidecar Injector|
|TCP|Inbound|9091||Sidecar Injector|