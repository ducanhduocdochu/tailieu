echo -e "192.168.100.105 k8s-master-1n192.168.100.106 k8s-master-2n192.168.100.107 k8s-master-3" | sudo tee -a /etc/hosts

sudo apt update -y && sudo apt upgrade -y

# phân quyền
adduser devops
visudo
su devops
cd /home/devops

# tắt swap, lưu cấu hình
sudo swapoff -a
sudo sed -i '/swap.img/s/^/#/' /etc/fsta

# cấu hình module kernel
echo -e "overlaynbr_netfilter" | sudo tee /etc/modules-load.d/containerd.conf > /dev/null

# tải module kernel
sudo modprobe overlay
sudo modprobe br_netfilter

# Cấu hình hệ thống mạng
echo "net.bridge.bridge-nf-call-ip6tables = 1" | sudo tee -a /etc/sysctl.d/kubernetes.conf
echo "net.bridge.bridge-nf-call-iptables = 1" | sudo tee -a /etc/sysctl.d/kubernetes.conf
echo "net.ipv4.ip_forward = 1" | sudo tee -a /etc/sysctl.d/kubernetes.conf

# Áp dụng cấu hình sysctl
sudo sysctl --system 

# Cài đặt các gói cần thiết và thêm kho Docker
sudo apt install -y curl gnupg2 software-properties-common apt-transport-https ca-certificates
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmour -o /etc/apt/trusted.gpg.d/docker.gpg
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
 
# Cài đặt containerd
sudo apt update -y
sudo apt install -y containerd.io

# Cấu hình containerd
containerd config default | sudo tee /etc/containerd/config.toml >/dev/null 2>&1
sudo sed -i 's/SystemdCgroup = false/SystemdCgroup = true/g' /etc/containerd/config.toml

# Khởi động containerd
sudo systemctl restart containerd
sudo systemctl enable containerd

# Install k8s
sudo mkdir -p /etc/apt/keyrings
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.30/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
echo "deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.30/deb/ /" | sudo tee /etc/apt/sources.list.d/kubernetes.list
sudo apt update

# Cài đặt các gói Kubernetes
sudo apt update -y
sudo apt install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl

# Khối lệnh reset cụm
sudo kubeadm reset -f
sudo rm -rf /var/lib/etcd
sudo rm -rf /etc/kubernetes/manifests/*

# Thực hiện trên server k8s-master-1
sudo kubeadm init --control-plane-endpoint "192.168.100.105:6443" --upload-certs
mkdir -p $HOME/.kube 
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config 
sudo chown $(id -u):$(id -g) $HOME/.kube/config
kubectl apply -f https://raw.githubusercontent.com/projectcalico/calico/v3.25.0/manifests/calico.yaml
 

# Thực hiện trên server k8s-master-2 và k8s-master-3
sudo kubeadm join 192.168.100.105:6443 --token your_token --discovery-token-ca-cert-hash your_sha --control-plane --certificate-key your_cert
mkdir -p $HOME/.kube 
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config 
sudo chown $(id -u):$(id -g) $HOME/.kube/config

# network
kubectl apply -f https://raw.githubusercontent.com/projectcalico/calico/v3.25.0/manifests/calico.yaml

# Cài đặt Helm
wget https://get.helm.sh/helm-v3.16.2-linux-amd64.tar.gz
tar xvf helm-v3.16.2-linux-amd64.tar.gz
sudo mv linux-amd64/helm /usr/bin/
helm version
 

# Cài đặt Ingress controller
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
helm repo update
helm pull ingress-nginx/ingress-nginx --version 4.11.3
tar -xzf ingress-nginx-4.11.3.tgz
vi ingress-nginx/values.yaml
>> Sửa type: LoadBalancing => type: NodePort
>> Sửa nodePort http: "" => http: "30080"
>> Sửa nodePort https: "" => https: "30443"
kubectl create ns ingress-nginx

# taint
kubectl taint nodes k8s-master-1 node-role.kubernetes.io/control-plane:NoSchedule-
kubectl taint nodes k8s-master-2 node-role.kubernetes.io/control-plane:NoSchedule-
kubectl taint nodes k8s-master-3 node-role.kubernetes.io/control-plane:NoSchedule-

helm -n ingress-nginx install ingress-nginx -f ingress-nginx/values.yaml ingress-nginx
