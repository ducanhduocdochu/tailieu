## Kiểm tra:
kubectl get nodes, kubectl get pods -A
## Các bước:
### Kiểm tra kubelet
systemctl status kubelet
journalctl -u kubelet -f
### Kiểm tra static pod manifest
/etc/kubernetes/manifests/kube-apiserver.yaml
cat /etc/kubernetes/manifests/kube-apiserver.yaml
### Xem container log
crictl ps | grep kube-apiserver
crictl logs <container-id>
## verify sau sửa
kubectl get nodes
kubectl cluster-info
