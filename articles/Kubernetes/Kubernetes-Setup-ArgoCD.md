Non-HA:
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/v2.9.3/manifests/install.yaml
HA:
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/v2.9.3/manifests/ha/install.yaml

```bash
kubectl get pods -n argocd -w
```


## Get the initial password:

Execute the following command, replacing <argocd-server-pod> with the name of your ArgoCD server pod:

```bash
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath='{.data.password}' | base64 --decode
```

```powershell
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath='{.data.password}' | ForEach-Object { [System.Text.Encoding]::UTF8.GetString([System.Convert]::FromBase64String($_)) }
```

## Port forwarding

```bash
kubectl port-forward svc/argocd-server -n argocd 8080:443
```


## Connect via argocdcli

Obtaining the ArgoCD server's hostname is also no big deal using:


```bash
kubectl get service argocd-server -n argocd --output=jsonpath='{.status.loadBalancer.ingress[0].hostname}'
```

```powershell
kubectl get service argocd-server -n argocd --output=jsonpath='{.status.loadBalancer.ingress[0].hostname}' | ForEach-Object { $_.Trim("'") }

```

And as the argocd login command has the parameters --username and --password, we can craft our login command like this:

argocd login $(kubectl get service argocd-server -n argocd --output=jsonpath='{.status.loadBalancer.ingress[0].hostname}') --username admin --password $(kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d; echo) --insecure






argocd admin initial-password -n argocd
Warning

You should delete the argocd-initial-admin-secret from the Argo CD namespace once you changed the password. The secret serves no other purpose than to store the initially generated password in clear and can safely be deleted at any time. It will be re-created on demand by Argo CD if a new admin password must be re-generated.

Using the username admin and the password from above, login to Argo CD's IP or hostname:


argocd login <ARGOCD_SERVER>
Note

The CLI environment must be able to communicate with the Argo CD API server. If it isn't directly accessible as described above in step 3, you can tell the CLI to access it using port forwarding through one of these mechanisms: 1) add --port-forward-namespace argocd flag to every CLI command; or 2) set ARGOCD_OPTS environment variable: export ARGOCD_OPTS='--port-forward-namespace argocd'.

Change the password using the command:


argocd account update-password

## Uninstall

```bash
kubectl delete -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/v2.9.3/manifests/ha/install.yaml

```