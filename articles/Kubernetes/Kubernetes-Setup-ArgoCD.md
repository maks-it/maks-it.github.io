# Setup ArgoCD to Kubernetes cluster

```bash
kubectl create namespace argocd-system
```



Non-HA (High Availability):

```bash
kubectl apply -n argocd-system -f https://raw.githubusercontent.com/argoproj/argo-cd/v2.9.3/manifests/install.yaml
```

HA:

```bash
kubectl apply -n argocd-system -f https://raw.githubusercontent.com/argoproj/argo-cd/v2.9.3/manifests/ha/install.yaml
```

```bash
kubectl logs argocd-redis-ha-server-1 -c config-init -n argocd-system
```

## Verify installation

```bash
kubectl get pods -n argocd-system -w
```

```bash
NAME                                               READY   STATUS    RESTARTS   AGE
argocd-application-controller-0                    1/1     Running   0          7m28s
argocd-applicationset-controller-5f975ff5-c84bb    1/1     Running   0          7m28s
argocd-dex-server-5cb44cbfcd-8npb7                 1/1     Running   0          7m28s
argocd-notifications-controller-566465df76-zl5cb   1/1     Running   0          7m28s
argocd-redis-69d46564c7-xwdnc                      1/1     Running   0          7m28s
argocd-repo-server-6d5f959b8f-hn8hb                1/1     Running   0          7m28s
argocd-server-7b6bb89949-8c5b2                     1/1     Running   0          7m28s
```

## Get the initial password:

Execute the following command, replacing <argocd-server-pod> with the name of your ArgoCD server pod:

```bash
kubectl -n argocd-system get secret argocd-initial-admin-secret -o jsonpath='{.data.password}' | base64 --decode
```

```powershell
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath='{.data.password}' | ForEach-Object { [System.Text.Encoding]::UTF8.GetString([System.Convert]::FromBase64String($_)) }
```

## Kubectl proxy

```bash
kubectl proxy
```

```
http://localhost:8001/api/v1/namespaces/argocd-system/services/https:argocd-server:https/proxy/
```

## Port forwarding

```bash
kubectl port-forward svc/argocd-server -n argocd-system 8080:443
```

## Uninstall

```bash
kubectl delete -n argocd-system -f https://raw.githubusercontent.com/argoproj/argo-cd/v2.9.3/manifests/ha/install.yaml
```