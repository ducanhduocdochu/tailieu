# Kiểm tra node
kubectl get nodes

# Kiểm tra kubelet
systemctl status kubelet

# Xem logs
journalctl -u kubelet

# Kiểm tra config
/var/lib/kubelet/config.yaml

# kiểm tra kubeconfig
/etc/kubernetes/kubelet.conf

# Sau sửa
systemctl restart kubelet

kubectl get nodes
