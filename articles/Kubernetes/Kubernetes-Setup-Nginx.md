
# Kubernetes Setup Nginx

[Nginx](https://bitnami.com/stack/nginx/helm)
[Nginx Chart Docs](https://github.com/bitnami/charts/tree/main/bitnami/nginx/#installing-the-chart)

Create nginx-system namespace on your kubernetes cluster

```bash
kubectl create namespace nginx-system
```

To specify that the Bitnami NGINX pods should only run on nodes k8swrk0001, k8swrk0002 and k8swrk0003, you can use a node selector.

```bash
kubectl label nodes k8swrk0001.corp.maks-it.com nginx=enabled
kubectl label nodes k8swrk0002.corp.maks-it.com nginx=enabled
kubectl label nodes k8swrk0003.corp.maks-it.com nginx=enabled
```


```yaml
replicaCount: 3
service:
  type: NodePort
  nodePorts:
    http: 30080
    https: 30443
extraEnvVars:
  - name: LOG_LEVEL
    value: error
image:
  debug: true
affinity:
  podAntiAffinity:
    requiredDuringSchedulingIgnoredDuringExecution:
    - labelSelector:
        matchLabels:
          app.kubernetes.io/name: nginx
          app.kubernetes.io/instance: RELEASE-NAME
      topologyKey: "kubernetes.io/hostname"
nodeSelector:
  nginx: enabled
```

```bash
helm install my-release oci://registry-1.docker.io/bitnamicharts/nginx \
  --namespace nginx-system \
  --values nginx-values.yml
```

```bash
kubectl get svc -n nginx-system
```

To access NGINX from outside the cluster, follow the steps below:

```bash
export NODE_PORT=$(kubectl get --namespace nginx-system -o jsonpath="{.spec.ports[0].nodePort}" services my-release-nginx)
export NODE_IP=$(kubectl get nodes --namespace nginx-system -o jsonpath="{.items[0].status.addresses[0].address}")
echo "http://${NODE_IP}:${NODE_PORT}"
```

```bash
helm delete my-release --namespace nginx-system
```
