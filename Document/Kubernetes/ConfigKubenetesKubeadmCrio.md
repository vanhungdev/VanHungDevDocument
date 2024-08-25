# Cài đặt Kubenetes

### Bước 1: Cài đặt cấu hịnh mạng

```bash

# Kubernetes Network Configuration

cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-iptables  = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward                 = 1
EOF

# Apply sysctl params without reboot
sudo sysctl --system


### Swap off
sudo swapoff -a
(crontab -l 2>/dev/null; echo "@reboot /sbin/swapoff -a") | crontab - || true

```


### Bước 2 cài container runtime cho Kubenetes:
```bash

### CRI-O
cat <<EOF | sudo tee /etc/modules-load.d/crio.conf
overlay
br_netfilter
EOF

sudo modprobe overlay
sudo modprobe br_netfilter

export OS="xUbuntu_20.04"
export VERSION="1.28"

cat <<EOF | sudo tee /etc/apt/sources.list.d/devel:kubic:libcontainers:stable.list
deb https://download.opensuse.org/repositories/devel:/kubic:/libcontainers:/stable/$OS/ /
EOF

cat <<EOF | sudo tee /etc/apt/sources.list.d/devel:kubic:libcontainers:stable:cri-o:$VERSION.list
deb http://download.opensuse.org/repositories/devel:/kubic:/libcontainers:/stable:/cri-o:/$VERSION/$OS/ /
EOF


curl -L https://download.opensuse.org/repositories/devel:kubic:libcontainers:stable:cri-o:$VERSION/$OS/Release.key | sudo apt-key --keyring /etc/apt/trusted.gpg.d/libcontainers.gpg add -

curl -L https://download.opensuse.org/repositories/devel:/kubic:/libcontainers:/stable/$OS/Release.key | sudo apt-key --keyring /etc/apt/trusted.gpg.d/libcontainers.gpg add -

sudo apt-get update
sudo apt-get install cri-o cri-o-runc cri-tools -y
sudo systemctl daemon-reload
sudo systemctl enable crio --now
```

### Bước 3 Cài đặt Kubeadm, kublet, kubectl:

```bash

sudo apt-get install -y apt-transport-https ca-certificates curl gpg
sudo mkdir -p -m 755 /etc/apt/keyrings

curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.31/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg

echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.31/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list

sudo apt-get update
sudo apt-get install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl
sudo systemctl enable --now kubelet
```

### Bước 4 Cài đặt cho master node
```bash
IPADDR="10.128.0.19" # Địa chỉ ip này là của vps google cloud Interal IP VPC network 
NODENAME=$(hostname -s) # Tên của node
POD_CIDR="192.168.0.0/16" # dãy mạng của container runtime

kubeadm init --apiserver-advertise-address=$IPADDR  --apiserver-cert-extra-sans=$IPADDR  --pod-network-cidr=$POD_CIDR --node-name $NODENAME --ignore-preflight-errors Swap



# Config để dùng lênh kubectl 
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config


# Kiểm tra đã sử dụng được chưa
kubectl get po -n kube-system
kubectl get nodes


# TRÊN WORKER NODE nếu muống dùng lệnh kubectl
# Cần tải file cấu hình từ master node về
mkdir -p /root/.kube
scp root@10.128.0.19:/root/.kube/config /root/.kube/config
```


### Bước 5: Cấu hình Network plugin

```bash
kubectl apply -f https://raw.githubusercontent.com/projectcalico/calico/v3.25.0/manifests/calico.yaml



```
Cấu Hình Network Plugin (như Calico) chỉ cần được áp dụng trên một node (thường là master node).  
Các worker node sẽ tự động nhận cấu hình và triển khai Network Plugin thông qua các tài nguyên Kubernetes.


### Bước 6: Join các worker node Chỉ chạy cho các worker node
```bash

# Join master node lênh này coppy khi chạy lênh kubeadm init ở bước 3
kubeadm join xxx:6443 --token xxx \
	--discovery-token-ca-cert-hash sha256:xxx


# Đánh label cho node
kubectl label node <worker-node-name>  node-role.kubernetes.io/worker=worker



```