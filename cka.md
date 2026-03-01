logs location in k8s:
/var/log/pods
/var/log/containers
crictl ps + crictl logs
docker ps + docker logs (in case when Docker is used)
kubelet logs: /var/log/syslog or journalctl

# 1
manifest kube-apiserver: /etc/kubernetes/manifests/kube-apiserver.yaml

backup: cp /etc/kubernetes/manifests/kube-apiserver.yaml ~/kube-apiserver.yaml.ori

watch crictl ps => theo dõi các pod
kubectl -n kube-system get pod => lấy các pod trong kube-system
cat /var/log/pods/kube-system_kube-apiserver-controlplane_a3a455d471f833137588e71658e739da/kube-apiserver/X.log
> 2022-01-26T10:41:12.401641185Z stderr F Error: unknown flag: --this-is-very-wrong  => lấy log trong pod

# 2
cat /var/log/syslog | grep kube-apiserver: lọc log liên quan đến kube-apiserver trong file syslog
/var/log/pods: thư mục chứa log của các Pod trên node

# 3
kubectl -n kube-system get pod
kubectl -n kube-system logs kube-controller-manager-controlplane

# 4 
ssh node01: ssh vào node
cat /var/log/syslog | grep kubelet: xem log node
service kubelet restart: restart kubelet
service kubelet status

# 5
k -n application1 get deploy: lấy deployment
k -n application1 logs deploy/api: lấy log deploy
k -n application1 describe deploy api: miêu tả deploy 
k -n application1 get cm: lấy configmap
k -n application1 edit deploy api: sửa deployment
