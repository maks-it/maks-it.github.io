helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update

kubectl create namespace prometheus
helm upgrade --install prometheus prometheus-community/prometheus --namespace prometheus



```bash
NAME                                                 READY   STATUS    RESTARTS   AGE
prometheus-alertmanager-0                            0/1     Pending   0          19m
prometheus-kube-state-metrics-6b464f5b88-ntjsp       1/1     Running   0          19m
prometheus-prometheus-node-exporter-d46c8            1/1     Running   0          18m
prometheus-prometheus-node-exporter-drg6r            1/1     Running   0          19m
prometheus-prometheus-node-exporter-glg58            1/1     Running   0          19m
prometheus-prometheus-node-exporter-hxgkf            1/1     Running   0          18m
prometheus-prometheus-node-exporter-j7wh2            1/1     Running   0          19m
prometheus-prometheus-node-exporter-zdt76            1/1     Running   0          18m
prometheus-prometheus-pushgateway-7857c44f49-t7tvm   1/1     Running   0          19m
prometheus-server-6b68fbd54b-7d7v8                   0/2     Pending   0          19m
```

Let's try to understand what's going wrong:

```bash
kubectl get pvc -n prometheus
```

```bash
NAME                                STATUS    VOLUME   CAPACITY   ACCESS MODES   STORAGECLASS   VOLUMEATTRIBUTESCLASS   AGE
prometheus-server                   Pending                                                     <unset>                 20m
storage-prometheus-alertmanager-0   Pending                                                     <unset>                 20m
```


```bash
kubectl describe pvc prometheus-server -n prometheus
kubectl describe pvc storage-prometheus-alertmanager-0 -n prometheus
```

```bash
kubectl get pvc prometheus-server -n dapr-system -o yaml
```

kubectl port-forward svc/prometheus-service -n prometheus 9091:9090 


```bash
helm uninstall prometheus -n prometheus
```