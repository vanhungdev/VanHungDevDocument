# Cài đặt Kubenetes

### Bước 1: Cài đặt cấu hình mạng

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


### Bước 2 cài container runtime, kubelet kubeadm kubectl cho Kubenetes:
```bash
# Set versions
export KUBERNETES_VERSION=v1.29
export CRIO_VERSION=v1.29

# Tạo thư mục cho keys
sudo mkdir -p /etc/apt/keyrings

# Thêm Kubernetes repository
sudo curl -fsSL https://pkgs.k8s.io/core:/stable:/${KUBERNETES_VERSION}/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg

echo "deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/${KUBERNETES_VERSION}/deb/ /" | sudo tee /etc/apt/sources.list.d/kubernetes.list

# Thêm CRI-O repository
sudo curl -fsSL https://pkgs.k8s.io/addons:/cri-o:/stable:/${CRIO_VERSION}/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/cri-o-apt-keyring.gpg

echo "deb [signed-by=/etc/apt/keyrings/cri-o-apt-keyring.gpg] https://pkgs.k8s.io/addons:/cri-o:/stable:/${CRIO_VERSION}/deb/ /" | sudo tee /etc/apt/sources.list.d/cri-o.list

# Cài đặt packages
sudo apt-get update
sudo apt-get install -y cri-o cri-o-runc cri-tools
sudo apt-get install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl

# Enable services
sudo systemctl daemon-reload
sudo systemctl enable crio --now
sudo systemctl enable kubelet --now


```

### Bước 4 Cài đặt cho master node
```bash

export IPADDR=$(hostname -I | awk '{print $1}') # Địa chỉ ip với máy ảo
export POD_CIDR="10.244.0.0/16" # dãy mạng của container runtime
export NODENAME=$(hostname -s) # Tên của node

sudo kubeadm init \
  --apiserver-advertise-address=${IPADDR} \
  --cri-socket=/var/run/crio/crio.sock \
  --apiserver-cert-extra-sans=${IPADDR} \
  --pod-network-cidr=${POD_CIDR} \
  --node-name ${NODENAME} \
  --ignore-preflight-errors Swap


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

# Cách lấy lại code join worker node
kubeadm token create --print-join-command
```

Sau khi triển khai ứng dụng dùng nodeport của service và External IP của service luôn:

```bash
root@instance-20240825-055816:~# kubectl get service
NAME                TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)        AGE
kubernetes          ClusterIP   10.96.0.1        <none>        443/TCP        172m
nginx-service       NodePort    10.103.177.161   <none>        80:32000/TCP   136m
spf-clean-service   NodePort    10.103.42.127    <none>        80:30213/TCP   4m26s

# Ví dụ VPS có IP như sau:

Internal IP           External IP
10.128.0.19           34.135.32.57

# Truy cập sẽ có url sau
http://35.239.213.160:30213/web-api/docs/index.html
```
