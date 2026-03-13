# Kiểm tra
kubectl get pods -n kube-system

kubectl get nodes

# Kiểm tra static pod
ls /etc/kubernetes/manifests xem có kube-controller-manager.yaml không

# Check container
crictl ps

crictl ps -a 

crictl logs <container-id>

vi /etc/kubernetes/manifests/kube-controller-manager.yaml

# Verify 
kubectl get pods -n kube-system

kubectl get nodes
