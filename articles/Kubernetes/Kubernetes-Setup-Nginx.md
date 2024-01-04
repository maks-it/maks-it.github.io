
# Kubernetes Setup Nginx

[Nginx](https://bitnami.com/stack/nginx/helm)
[Nginx Chart Docs](https://github.com/bitnami/charts/tree/main/bitnami/nginx/#installing-the-chart)

Create nginx-system namespace on your kubernetes cluster

```bash
kubectl create namespace nginx-system
```

```yaml
service:
  type: NodePort
  port: 80
```


```bash
helm install my-release oci://registry-1.docker.io/bitnamicharts/nginx \
  --namespace nginx-system \
  --values nginx-values.yml
```





```bash
helm delete my-release --namespace nginx-system
```