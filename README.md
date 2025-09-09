
Node.js app that auto-scales on Kubernetes.

```bash
kubectl apply -f k8s/
hey -z 5m -q 100 -c 50 http://$(kubectl get svc hello-service -o jsonpath='{.status.loadBalancer.ingress[0].ip}')
kubectl get hpa -w

Scales from 1 → 5 pods under load
