
Node.js app that auto-scales on Kubernetes.

```bash
kubectl apply -f k8s/
hey -z 5m -q 100 -c 50 http://$(kubectl get svc hello-service -o jsonpath='{.status.loadBalancer.ingress[0].ip}')
kubectl get hpa -w

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

`
