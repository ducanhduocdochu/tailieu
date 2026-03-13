# Kiểm tra
kubectl get nodes

# Các bước
## Kiểm tra static pod
ls /etc/kubernetes/manifests

## Check container
crictl ps
crictl ps -a
crictl logs <container-id>

# Verify 
kubectl get nodes
kubectl cluster-info
ls /etc/kubernetes/manifests
systemctl status kubelet
journalctl -u kubelet
