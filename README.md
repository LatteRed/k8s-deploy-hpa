
Node.js app that auto-scales on Kubernetes.

```bash
kubectl apply -f k8s/
hey -z 5m -q 100 -c 50 http://$(kubectl get svc hello-service -o jsonpath='{.status.loadBalancer.ingress[0].ip}')
kubectl get hpa -w

when monitoring for hpa sometimes the monitoring server needs to be installed:

kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml

then patched:

kubectl patch deploy -n kube-system metrics-server \
  --type 'json' \
  -p='[{"op": "add", "path": "/spec/template/spec/containers/0/args", "value": [
    "--kubelet-insecure-tls",
    "--kubelet-preferred-address-types=InternalIP"
  ]}]'

use this when the monitoring server seems a bit wonky or doesnt return actul data or numbers.

Scales from 1 → 5 pods under load

Instal Helm and then make some monitoring.

helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update
kubectl create namespace monitoring
helm install prometheus prometheus-community/kube-prometheus-stack --namespace monitoring
kubectl port-forward -n monitoring svc/prometheus-grafana 3000:80

or get a public ip...

point to a load balencer instead

kubectl patch svc -n monitoring prometheus-grafana \
  --type='json' \
  -p='[{"op": "replace", "path": "/spec/type", "value": "LoadBalancer"}]'

then a wait a but for IP to be provisioned.

you can monitoring with the following:


```
kubectl get svc -n monitoring prometheus-grafana
```

you  should get something like: 

```
prometheus-grafana     LoadBalancer   10.245.0.123   45.33.201.123   80:32123/TCP
```


you can perform the same steps to expose promethues but you should have to. actually kinda dangerous.

The user is admin and the password id ive to you in a prompt when installing grafana with helm.

From there you can make a few dashboards and then bring your up node app and test load on the app and watch for auto scaling.


for prebuilt dashabaords, the json I left in prebuilt folder.

updated added argoCD

kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml


port forward so you can see it.

kubectl port-forward svc/argocd-server -n argocd 8080:443


Also when fixing the hpa stats you will sometimes get unknown can be fix with the follwing below.


kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
kubectl patch deploy -n kube-system metrics-server \
  --type 'json' \
  -p='[{"op": "add", "path": "/spec/template/spec/containers/0/args", "value": [
    "--kubelet-insecure-tls",
    "--kubelet-preferred-address-types=InternalIP"
  ]}]'

final teardown.


kubectl delete -f k8s/
helm uninstall prometheus -n monitoring
kubectl delete namespace monitoring
kubectl delete namespace argocd


dont forget that when using argo you can get passwords from the follwing:
kubectl get secret -n argocd argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d


