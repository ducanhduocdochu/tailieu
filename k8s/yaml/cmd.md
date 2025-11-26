# ✅ 1. Cluster info
```
kubectl version --short
kubectl cluster-info
kubectl get componentstatuses   # kiểm tra health control plane
kubectl api-resources           # liệt kê tài nguyên hỗ trợ
kubectl api-versions
```

# ✅ 2. Get – liệt kê resource
```
kubectl get nodes
kubectl get pods -A
kubectl get svc -A
kubectl get deployments -A
kubectl get ns
```

## dạng wide để xem IP, node, port
```
kubectl get pods -o wide
kubectl get nodes -o wide
```

# ✅ 3. Describe – xem chi tiết
```
kubectl describe pod <pod>
kubectl describe node <node>
kubectl describe deployment <deploy>
kubectl describe svc <svc>
```

# ✅ 4. Logs
```
kubectl logs <pod>
kubectl logs <pod> -f              # tail log
kubectl logs <pod> -c <container>  # log container cụ thể
```

# ✅ 5. Exec vào pod
```
kubectl exec -it <pod> -- /bin/bash
kubectl exec -it <pod> -- sh
```

# ✅ 6. Apply / Delete
```
kubectl apply -f app.yaml
kubectl delete -f app.yaml

kubectl delete pod <pod>
kubectl delete deployment <deploy>
kubectl delete svc <svc>
kubectl delete ns <namespace>
```

# ✅ 7. Edit trực tiếp
```
kubectl edit deployment <deploy>
kubectl edit svc <svc>
kubectl edit configmap <cm>
kubectl edit secret <secret>
```

# ✅ 8. Scale
```
kubectl scale deployment <deploy> --replicas=3
kubectl rollout restart deployment <deploy>
```

# ✅ 9. Rollout (triển khai / rollback)
```
kubectl rollout status deployment <deploy>
kubectl rollout history deployment <deploy>
kubectl rollout undo deployment <deploy>
```

# ✅ 10. Port-forward
```
kubectl port-forward pod/<pod> 8080:80
kubectl port-forward svc/<svc> 8080:80
```

# ✅ 11. ConfigMap / Secret

ConfigMap:
```
kubectl create configmap my-config --from-literal=KEY=value
kubectl create configmap my-config --from-file=config.yaml
```

Secret:
```
kubectl create secret generic my-secret --from-literal=password=123456
kubectl create secret tls my-tls --cert=cert.crt --key=cert.key
```

# ✅ 12. Context / Namespace
```
kubectl config get-contexts
kubectl config use-context <ctx>
kubectl config set-context --current --namespace=<ns>

kubectl get pods -n <ns>
```

# ✅ 13. Apply Kustomize
```
kubectl apply -k ./kustomize-folder
```

# ✅ 14. Debug (troubleshoot)
```
kubectl describe pod <pod>
kubectl logs <pod>
kubectl get events --sort-by=.metadata.creationTimestamp
kubectl top nodes
kubectl top pods -A
```
