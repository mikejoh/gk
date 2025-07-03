# Install Argo Workflows on `kind`

1. Create a cluster with `kind`.

2. Install Argo Workflows:

```bash
helm upgrade --install -f argo-workflows-values.yaml --create-namespace --namespace argo-workflows argo-workflows argo/argo-workflows
```

3. Port forward to the argo-server:

```bash
kubectl port-forward -n argo-workflows svc/argo-workflows-server 8080:2746
```

3. Create a service account to log in:

```bash
ARGO_TOKEN="Bearer $(kubectl get secret -n argo-workflows wf-user-token -o=jsonpath='{.data.token}' | base64 --decode)"
echo $ARGO_TOKEN
```

4. Browse to the Argo Workflows UI!
