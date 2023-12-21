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

```bash
helm upgrade --install dapr dapr/dapr --version=1.9 --namespace dapr-system
```

or for High Availability setup

```bash
helm upgrade --install dapr dapr/dapr --version=1.12 --namespace dapr-system --set global.ha.enabled=true
```

in case you want to use `values.yml`, create `values.yml` with following content

```yml
global:
  ha:
    enabled: true
```

```bash
helm install dapr dapr/dapr --version=1.12 --namespace dapr-system --values values.yml
```


## Install dapr dashboard

```bash
helm upgrade --install dapr-dashboard dapr/dapr-dashboard --version=0.13 --namespace dapr-system
```

## Verify installation

Once the chart installation is complete verify the dapr-operator, dapr-placement, dapr-sidecar-injector and dapr-sentry pods are running in the dapr-system namespace:

```bash
kubectl get pods -n dapr-system -w
```

```bash
NAME                                     READY     STATUS    RESTARTS   AGE
dapr-dashboard-7bd6cbf5bf-xglsr          1/1       Running   0          40s
dapr-operator-7bd6cbf5bf-xglsr           1/1       Running   0          40s
dapr-placement-7f8f76778f-6vhl2          1/1       Running   0          40s
dapr-sidecar-injector-8555576b6f-29cqm   1/1       Running   0          40s
dapr-sentry-9435776c7f-8f7yd             1/1       Running   0          40s
```

## Uninstall Dapr on Kubernetes

```bash
helm uninstall dapr -n dapr-system
helm uninstall dapr-dashboard -n dapr-system
```

## Port forwarding

```bash
kubectl port-forward svc/dapr-dashboard -n dapr-system 8081:8080
```




kubectl -n dapr-system get pvc raft-log-dapr-placement-server-0 -o yaml
kubectl describe pvc raft-log-dapr-placement-server-0 -n dapr-system

kubectl get pv
kubectl get storageclass
kubectl get pvc -A

dapr status -k
kubectl get all -n dapr-system

kubectl replace -f https://raw.githubusercontent.com/dapr/dapr/v1.12.2/charts/dapr/crds/httpendpoints.yaml