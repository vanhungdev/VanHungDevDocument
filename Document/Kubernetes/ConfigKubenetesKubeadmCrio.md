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
IPADDR="External IP" # Địa chỉ ip này là của vps google  cloud Interal IP VPC network 
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

## Bash script cài tự động

```bash
#!/bin/bash

LOG_FILE="/var/log/ssh_setup.log"

# Ghi log với kết quả ngắn gọn khi chạy lệnh
log_result() {
    if [ $? -eq 0 ]; then
        echo "$(date +'%Y-%m-%d %H:%M:%S') - $1: Thành công" | sudo tee -a $LOG_FILE
    else
        echo "$(date +'%Y-%m-%d %H:%M:%S') - $1: Thất bại" | sudo tee -a $LOG_FILE
    fi
}


sudo cd >> $LOG_FILE 2>&1
sudo cd /root >> $LOG_FILE 2>&1

sudo pwd >> $LOG_FILE 2>&1
sudo ls -a >> $LOG_FILE 2>&1

log_result "Bắt đầu cài đặt và cấu hình SSH server"

# Cài đặt OpenSSH server
sudo apt-get update >> $LOG_FILE 2>&1
sudo apt-get install -y openssh-server >> $LOG_FILE 2>&1
log_result "Cài đặt OpenSSH server"

# Sao lưu file cấu hình gốc
sudo cp /etc/ssh/sshd_config /etc/ssh/sshd_config.backup >> $LOG_FILE 2>&1
log_result "Sao lưu file cấu hình"

# Chỉnh sửa file cấu hình bằng sed
sudo sed -i \
    -e 's/^#Port 22/Port 22/' \
    -e 's/^#ListenAddress 0.0.0.0/ListenAddress 0.0.0.0/' \
    -e 's/^#PermitRootLogin.*/PermitRootLogin yes/' \
    -e 's/^#PubkeyAuthentication.*/PubkeyAuthentication yes/' \
    -e 's|^#AuthorizedKeysFile.*|AuthorizedKeysFile .ssh/authorized_keys .ssh/authorized_keys2|' \
    -e 's/^#PasswordAuthentication.*/PasswordAuthentication yes/' \
    -e 's/^#UsePAM.*/UsePAM yes/' \
    -e 's/^#X11Forwarding.*/X11Forwarding yes/' \
    -e 's/^#PrintMotd.*/PrintMotd no/' \
    -e '/^ChallengeResponseAuthentication /s/^/#/' \
    -e 's|^#Subsystem.*|Subsystem sftp /usr/lib/openssh/sftp-server|' \
    /etc/ssh/sshd_config >> $LOG_FILE 2>&1
log_result "Chỉnh sửa file cấu hình"

# Tạo thư mục .ssh nếu chưa tồn tại
sudo mkdir -p /root/.ssh
sudo chmod 700 /root/.ssh
log_result "Tạo thư mục /root/.ssh"

# Tạo key SSH nếu chưa có
cd /root/.ssh || exit
if [ ! -f id_rsa ]; then
    sudo ssh-keygen -t rsa -b 4096 -N "" -f id_rsa >> $LOG_FILE 2>&1
    log_result "Tạo SSH key"
else
    log_result "SSH key đã tồn tại"
fi

# Sao chép public key vào authorized_keys
sudo cp id_rsa.pub authorized_keys >> $LOG_FILE 2>&1
log_result "Sao chép public key vào authorized_keys"

# Hiển thị nội dung private key
sudo cat id_rsa >> $LOG_FILE 2>&1

# Cấp quyền cho file authorized_keys
chmod 600 /root/.ssh/authorized_keys
chmod 700 /root/.ssh
chmod 700 /root

# Khởi động lại dịch vụ SSH để áp dụng cấu hình mới
sudo systemctl restart ssh >> $LOG_FILE 2>&1
log_result "Khởi động lại dịch vụ SSH"

log_result "Hoàn tất cấu hình SSH server"


# STEP 0
#####################

#### Network prep
log_result "Bắt đầu cấu hình mạng"
cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-iptables  = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward                 = 1
EOF

# Apply sysctl params without reboot
sudo sysctl --system >> $LOG_FILE 2>&1
log_result "Cấu hình mạng đã được áp dụng"

### Swap off
log_result "Vô hiệu hóa swap"
sudo swapoff -a >> $LOG_FILE 2>&1
(crontab -l 2>/dev/null; echo "@reboot /sbin/swapoff -a") | crontab - || true >> $LOG_FILE 2>&1
log_result "Swap đã được vô hiệu hóa"

# STEP 1
#####################

### CRI-O
log_result "Bắt đầu cài đặt CRI-O"
cat <<EOF | sudo tee /etc/modules-load.d/crio.conf
overlay
br_netfilter
EOF

sudo modprobe overlay >> $LOG_FILE 2>&1
sudo modprobe br_netfilter >> $LOG_FILE 2>&1

export OS="xUbuntu_20.04" >> $LOG_FILE 2>&1
export VERSION="1.28" >> $LOG_FILE 2>&1

cat <<EOF | sudo tee /etc/apt/sources.list.d/devel:kubic:libcontainers:stable.list
deb https://download.opensuse.org/repositories/devel:/kubic:/libcontainers:/stable/$OS/ /
EOF

cat <<EOF | sudo tee /etc/apt/sources.list.d/devel:kubic:libcontainers:stable:cri-o:$VERSION.list
deb http://download.opensuse.org/repositories/devel:/kubic:/libcontainers:/stable:/cri-o:/$VERSION/$OS/ /
EOF

curl -L https://download.opensuse.org/repositories/devel:kubic:libcontainers:stable:cri-o:$VERSION/$OS/Release.key | sudo apt-key --keyring /etc/apt/trusted.gpg.d/libcontainers.gpg add -
curl -L https://download.opensuse.org/repositories/devel:/kubic:/libcontainers:/stable/$OS/Release.key | sudo apt-key --keyring /etc/apt/trusted.gpg.d/libcontainers.gpg add -

sudo apt-get update >> $LOG_FILE 2>&1
sudo apt-get install cri-o cri-o-runc cri-tools -y >> $LOG_FILE 2>&1

sudo systemctl daemon-reload >> $LOG_FILE 2>&1
sudo systemctl enable crio --now >> $LOG_FILE 2>&1
#sudo systemctl status crio
log_result "Cài đặt CRI-O hoàn tất"

# STEP 2
#####################

### Kubeadm, kubelet, kubectl
log_result "Bắt đầu cài đặt kubeadm, kubelet, kubectl"
sudo apt-get update >> $LOG_FILE 2>&1

sudo apt-get install -y apt-transport-https ca-certificates curl gpg >> $LOG_FILE 2>&1

sudo mkdir -p -m 755 /etc/apt/keyrings >> $LOG_FILE 2>&1

curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.31/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg

echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.31/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list

sudo apt-get update >> $LOG_FILE 2>&1

sudo apt-get install -y kubelet kubeadm kubectl >> $LOG_FILE 2>&1

sudo apt-mark hold kubelet kubeadm kubectl >> $LOG_FILE 2>&1

sudo systemctl enable --now kubelet >> $LOG_FILE 2>&1

log_result "Cài đặt kubeadm, kubelet, kubectl hoàn tất"
```

Đối với master node cần chạy thêm `Bước 4` worker node thì join và coppy file về.
