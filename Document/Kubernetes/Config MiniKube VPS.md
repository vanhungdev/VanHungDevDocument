# Cấu hình MiniKube và systemctl

Chạy lần lượt các lệnh sau:

```bash
# Download the latest release with the command
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
```

Cấp quyền và kiểm tra version

```bash
   chmod +x kubectl
   sudo mv ./kubectl /usr/local/bin/kubectl
   kubectl version
```


MiniKube
```bash
  grep -E --color 'vmx|svm' /proc/cpuinfo

  curl -Lo minikube https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64   && chmod +x minikube

  sudo mkdir -p /usr/local/bin/
  sudo install minikube /usr/local/bin/

  # Start minikube node
  minikube start --driver=docker --force

  # Nếu lỗi không có quyền
  rm /tmp/juju-*
  minikube delete
  sudo usermod -aG docker root

  # Kiểm tra trạng thái
  minikube status

  # Xem ndoe
  kubectl get nodes

  # Cài đặt Xem CPU, RAM trên dashboard
  minikube addons enable metrics-server
   
  # Khởi chạy tạo DashBoard
  minikube dashboard

```

Truy cập Dashboard bằng proxy
```bash
  # Nếu chỉ xem một chút thì allow proxy cho nó
  kubectl proxy --address=0.0.0.0 --accept-hosts='.*'

  http://34.29.18.204:8001/api/v1/namespaces/kubernetes-dashboard/services/http:kubernetes-dashboard:/proxy/#/about?namespace=default

```

### Truy cập Dashboard bằng service node port
```bash
  # Xem thấy cả service
  k get service -n kubernetes-dashboard


  # Kết quả
  [root@instance-20240521-043352 ~]# k get service -n kubernetes-dashboard
  NAME                        TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
  dashboard-metrics-scraper   ClusterIP   10.109.109.3    <none>        8000/TCP       12m
  kubernetes-dashboard        NodePort    10.101.16.230   <none>        80:32507/TCP   12m
```

Bây giờ chúng ta cần chỉnh Service có tên kubernetes-dashboard từ `ClusterIP` thành `NodePort` là có thể truy cập được.

```bash
# Edit một service có sẵn
kubectl edit svc kubernetes-dashboard -n kubernetes-dashboard
```
  Bấm `i` để có thể chỉnh sửa, bấm `Esc` rồi `wq` để lư và thoát. Vậy là service đã được chỉnh sửa

Kiểm tra ip của `minikube`
```bash
minikube ip
```

Xen tất cả danh sách service đang chạy của ndoe `minikube`
```bash
minikube service list

[root@instance-20240521-043352 ~]# minikube service list

#Kết quả
|----------------------|---------------------------|--------------|---------------------------|
|      NAMESPACE       |           NAME            | TARGET PORT  |            URL            |
|----------------------|---------------------------|--------------|---------------------------|
| default              | kubernetes                | No node port |                           |
| kube-system          | kube-dns                  | No node port |                           |
| kube-system          | metrics-server            | No node port |                           |
| kubernetes-dashboard | dashboard-metrics-scraper | No node port |                           |
| kubernetes-dashboard | kubernetes-dashboard      |           80 | http://192.168.49.2:32507 |
|----------------------|---------------------------|--------------|---------------------------|
```

Bây giờ chúng ta cần ping xem `kubernetes-dashboard` đã hoạt động chưa?

```bash
# Lệnh curl -I chỉ lấy phần header
curl -I  http://192.168.49.2:32507

# Kết quả
[root@instance-20240521-043352 ~]# curl -I  http://192.168.49.2:32507
HTTP/1.1 200 OK
Accept-Ranges: bytes
Cache-Control: no-cache, no-store, must-revalidate
Content-Length: 1412
Content-Type: text/html; charset=utf-8
Last-Modified: Fri, 16 Sep 2022 11:49:34 GMT
Date: Fri, 31 May 2024 15:32:09 GMT

```

Như vậy là đã public được minikube dashboard. Nhưng làm sau có thể truy cập được từ Extenal IP của VPS:

Bây giờ chúng ta fw port như sau:
```bash
sudo iptables -t nat -A PREROUTING -p tcp --dport 32507 -j DNAT --to-destination <Minikube IP>:32507


sudo iptables -t nat -A PREROUTING -p tcp --dport 8001 -j DNAT --to-destination 192.168.49.2:32603
sudo iptables -t nat -A PREROUTING -p tcp --dport 30088 -j DNAT --to-destination 192.168.49.2:30088

# Nếu chưa truy cập được thì disale firewall
sudo ufw status
sudo ufw disable
sudo ufw status

```

### Cài đặt rancher:
Chạy docker compose như sau:

 ```bash

version: '3'
services:
  rancher:
    image: rancher/rancher:latest
    container_name: rancher
    restart: unless-stopped
    privileged: true
    ports:
      - "8080:80"   # Thay đổi 8080 thành port bạn muốn sử dụng cho HTTP
      - "8443:443"  # Thay đổi 8443 thành port bạn muốn sử dụng cho HTTPS
    volumes:
      - rancher-data:/var/lib/rancher

volumes:
  rancher-data:


# Xem cấu hình của cluster và ip api server
kubectl cluster-info

# Dổi IP thành Internal IP
curl --insecure -sfL https://<Internal IP>:8443/v3/import/9pp5ct65lnqh4rqdw2cqfh8kj5tftkm8l6nf64tlmrf7jcmkz56jjt_c-m-bnp5kzxj.yaml | kubectl apply -f -
#
 ```
Xem cấu hình minikube 

```bash
# Xem cấu hình và IP minikube
minikube kubectl -- config view --raw

# Vào link sau
https://34.135.32.57:8443/dashboard/c/_/manager/provisioning.cattle.io.cluster/create?mode=import


chọn Inport any.kubernetes cluter > Generic

# chạy script sau:
curl --insecure -sfL https://34.135.32.57:8443/v3/import/hdd2pn88tb7bn87bql7tffvj2bdpfhsfxltdnm5ghvxhbzs5t469hf_c-m-fssj6jdw.yaml | kubectl apply -f - --debug


```

Chỉ là mẫu cài theo link của rancher mới đúng



### Tạo minikube khởi động cùng server:
```bash

# Allow quyền chạy minikube
sudo usermod -aG docker root

# Tạo file bash script
sudo nano /usr/local/bin/start-minikube.sh

```


Điền vào nội dung file:
```bash
#!/bin/bash

# Định nghĩa file log
LOG_FILE="/var/log/minikube_setup.log"

# Hàm ghi log
log_command() {
    echo "$(date '+%Y-%m-%d %H:%M:%S') - Lệnh: $1" >> "$LOG_FILE"
    output=$($1 2>&1)
    echo "Kết quả:" >> "$LOG_FILE"
    echo "$output" >> "$LOG_FILE"
    echo "---" >> "$LOG_FILE"
}

# Bắt đầu script
echo "$(date '+%Y-%m-%d %H:%M:%S') - Bắt đầu cài đặt Minikube" >> "$LOG_FILE"

# Cấp quyền docker root

log_command "sudo usermod -aG docker root"

# Khởi động Minikube
log_command "minikube start --driver=docker --force"

# Vô hiệu hóa UFW
log_command "sudo ufw disable"

# Thêm quy tắc NAT với iptables
# Minikube Dashboard khởi động với nodePort 30000
log_command "sudo iptables -t nat -A PREROUTING -p tcp --dport 8001 -j DNAT --to-destination 192.168.49.2:30000"

# Application khởi động với bắt đầu từ số 1
log_command "sudo iptables -t nat -A PREROUTING -p tcp --dport 30001 -j DNAT --to-destination 192.168.49.2:30001"

# More application
# Thêm các lệnh khác ở đây và sử dụng log_command để ghi log

echo "$(date '+%Y-%m-%d %H:%M:%S') - Cài đặt hoàn tất" >> "$LOG_FILE"

```
Cấu hình service cho minikube:
```bash
sudo nano /etc/systemd/system/minikube.service
```


Bỏ nội dung vào file:
```bash
[Unit]
Description=Start Minikube
After=docker.service
Wants=network-online.target
After=network-online.target

[Service]
Type=oneshot
ExecStart=/usr/local/bin/start-minikube.sh
RemainAfterExit=true
User=root

[Install]
WantedBy=multi-user.target


```


Chạy các script sau

```bash

# Stop minikube hiện tại đang chạy để test script
minikube stop

# Khởi tạo
# Sau khi sửa file cần chạy lại lệnh này
sudo systemctl daemon-reload

sudo systemctl enable minikube.service

# Xem thử đã chạy được chưa
sudo systemctl start minikube.service


# Kiểm tra trạng thái minikube
minikube status

# Xem log
sudo cat /var/log/minikube_setup.log
```



Xem log minikube start cùng server:

```bash
# Xem trạng thái dịch vụ
sudo systemctl status minikube.service

# Xem log
sudo journalctl -u minikube.service
```

 Thực hiện chức năng DNAT (Destination Network Address Translation) trong iptables, một công cụ tường lửa phổ biến trên Linux. Cụ thể, lệnh này sẽ chuyển hướng lưu lượng truy cập TCP đến trên cổng 32507 của máy chủ (VPS) của bạn đến cổng 32507 của một máy chủ khác có địa chỉ IP là 192.168.49.2.


Như vậy là từ IP của VPS chúng ta có thể truy cập được Dashboard của minikube.

```bash
http://<VPS IP>:32507

```

Update Image của một file deployment (Có thể dùng để CICD)

```Bash
[root@instance-20240524-042138 ~]# kubectl set image deployment/medical-examination-api medical-examination-api=vanhungdev/bloghung:5f5edeea
deployment.apps/medical-examination-api image updated
```


### CICD
CICD với deployment:  
```bash
      kubectl set image deplyment/medical-examination-api medical-examination-api=$DOCKER_IMAGE
      kubectl describe deployment/medical-examination-api
      kubectl get pods
```

CICD với StatefulSets:
```bash
      echo "======================================================================================"
      kubectl get pods
      echo "--------------------------------------------------------------------------------------"
      kubectl set image statefulset.apps/blog-hung-mvc blog-hung-mvc=$DOCKER_IMAGE
      echo "--------------------------------------------------------------------------------------"
      kubectl describe statefulset.apps/blog-hung-mvc
      echo "--------------------------------------------------------------------------------------"
      kubectl get pods
```



History thao tác với CKE Goole Kubernetes Engine:

```bash
    1  gcloud container clusters get-credentials autopilot-cluster-1 --region us-central1 --project coastal-campus-428405-c4
    2  kubectl get pods
    3  kubectl get node
    4  kubectl get nodes
    5  kubectl version
    6  kubectl create deployment hello-world-rest-api --image=in28min/hello-world-rest-api:0.0.1.RELEASE
    7  kubectl expose deployment hello-world-rest-api --type=LoadBalancer --port=8080
    8  kubectl scale deployment hello-world-rest-api --replicas=3
    9  kubectl get pods
   10  kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.0.0/aio/deploy/recommended.yaml
   11  vim dashboard-adminuser.yaml
   12  kubectl apply -f   dashboard-adminuser.yaml
   13  vim ClusterRoleBinding.yml
   14  k apply -f  ClusterRoleBinding.yml
   15  kubectl apply -f  ClusterRoleBinding.yml
   16  vim ClusterRoleBinding.yml
   17  kubectl apply -f  ClusterRoleBinding.yml
   18  kubectl -n kubernetes-dashboard create token admin-user
   19  kubectl get all
   20  kubectl -n kubernetes-dashboard create token admin-user
   21  dashboard-adminuser.yaml
   22  vim dashboard-adminuser.yaml
   23  kubectl apply -f   dashboard-adminuser.yaml
   24  kubectl -n kubernetes-dashboard create token admin-user
   25  gcloud container clusters get-credentials autopilot-cluster-1 --region us-central1 --project coastal-campus-428405-c4
   26  kubectl get node
   27  gcloud container clusters get-credentials autopilot-cluster-1 --region us-central1 --project coastal-campus-428405-c4
   28  kubectl get ndoe
   29  kubectl get node
   30  k get all -n kubernetes-dashboard
   31  clear
   32  kubectl get services -n kubernetes-dashboard
   33  kubectl get pods -n kubernetes-dashboard
   34  kubectl get deployment -n kubernetes-dashboard
   35  kubectl get replicaset -n kubernetes-dashboard
   36  clear
   37  kubectl edit service kubernetes-dashboard -n kubernetes-dashboard
   38  k get service
   39  kubectl get servioce
   40  kubectl get service
   41  kubectl get service  -n kubernetes-dashboard
   42  kubectl edit service kubernetes-dashboard -n kubernetes-dashboard
   43  gcloud container clusters get-credentials autopilot-cluster-1 --region us-central1 --project coastal-campus-428405-c4
   44  kubectl get svc
   45  kubectl edit service hello-world-rest-api
   46  kubectl edit service kubernetes-dashboard -n kubernetes-dashboard
   47  k get pods -n kubernetes-dashboard
   48  kubectl  get pods -n kubernetes-dashboard
   49  kubectl describe kubernetes-dashboard-7b67d5ddb7-lc2gx
   50  kubectl describe kubernetes-dashboard-7b67d5ddb7-lc2gx -n kubernetes-dashboard
   51  kubectl describe pods kubernetes-dashboard-7b67d5ddb7-lc2gx -n kubernetes-dashboard
   52  Dựa vào thông tin bạn cung cấp, tôi thấy Kubernetes Dashboard đang chạy với port 8443. Để thay đổi port này, bạn cần thực hiện một số bước:
   53  Chỉnh sửa Deployment:
   54  Bạn cần chỉnh sửa Deployment của Kubernetes Dashboard để thay đổi port mà container lắng nghe. Tuy nhiên, việc này có thể gây ra vấn đề với cấu hình hiện tại và chứng chỉ SSL.
   55  Thay đổi Service:
   56  Thay vì thay đổi port trong Deployment, bạn có thể giữ nguyên port 8443 của container và chỉ thay đổi port trong Service. Đây là cách an toàn hơn.
   57  Để thay đổi port trong Service, hãy thực hiện các bước sau:
   58  Copykubectl edit service kubernetes-dashboard -n kubernetes-dashboard
   59  kubectl edit service kubernetes-dashboard -n kubernetes-dashboard
   60  kubectl edit pods kubernetes-dashboard -n kubernetes-dashboard
   61  clear
   62  kubectl edit pods kubernetes-dashboard-7b67d5ddb7-lc2gx -n kubernetes-dashboard
   63  kubectl edit pods kubernetes-dashboard -n kubernetes-dashboard
   64  kubectl edit pods kubernetes-dashboard-7b67d5ddb7-lc2gx -n kubernetes-dashboard
   65  gcloud container clusters get-credentials autopilot-cluster-1 --region us-central1 --project coastal-campus-428405-c4
   66  kubectl edit pods kubernetes-dashboard-7b67d5ddb7-lc2gx -n kubernetes-dashboard
   67  kubectl get all -kubernetes-dashboard
   68  kubectl get all -n kubernetes-dashboard
   69  kubectl describe deployment.apps/kubernetes-dashboard
   70  kubectl describe deployment.apps/kubernetes-dashboard -n kubernetes-dashboard
   71  period=10s
   72  kubectl get deployment kubernetes-dashboard -n kubernetes-dashboard -o yaml > dashboard-deployment.yaml
   73  vi dashboard-deployment.yaml
   74  kubectl delete namespace kubernetes-dashboard
   75  vi dashboard-http.yaml
   76  clear
   77  kubectl get all -n kubernetes-dashboard
   78  clear
   79  kubectl -n kubernetes-dashboard create token admin-user
   80  git clone
   81  clear
   82  git clone https://github.com/vanhungdev/KubernetesYAMLFile.git
   83  ls
   84  cd KubernetesYAMLFile
   85  ls
   86  git pull
   87  ls
   88  cd Dashboard
   89  ls
   90  vi dashboard.yaml
   91  kubectl apply -f dashboard.yaml 
   92  kubectl get all -n kubernetes-dashboard
   93  alias k=kubectl
   94  k get pods
   95  k describe pod/kubernetes-dashboard-ddc74bc-jqg5f
   96  k describe pod/kubernetes-dashboard-ddc74bc-jqg5f -n kubernetes-dashboard
   97  kubectl get all -n kubernetes-dashboard
   98  k describe service/kubernetes-dashboard
   99  k describe service/kubernetes-dashboard -n kubernetes-dashboard
  100  clear
  101  kubectl edit service kubernetes-dashboard -n kubernetes-dashboard
  102  clear
  103  git pull
  104  ls
  105  kubectl -n kubernetes-dashboard create token admin-user
  106  k apply -f dashboard-adminuser.yaml
  107  kubectl -n kubernetes-dashboard create token admin-user
  108  gcloud container clusters get-credentials autopilot-cluster-1 --region us-central1 --project coastal-campus-428405-c4
  109  kubectl -n kubernetes-dashboard create token kubernetes-dashboard
  110  kubectl get all -n kubernetes-dashboard
  111  k get Secret
  112  kubectl get Secret -kubernetes-dashboard
  113  kubectl get Secret -n kubernetes-dashboard
  114  kubectl -n kubernetes-dashboard delete serviceaccount admin-user
  115  kubectl -n kubernetes-dashboard delete clusterrolebinding admin-user
  116  clear
  117  kubectl delete -n kubernetes-dashboard
  118  kubectl delete namespace kubernetes-dashboard
  119  kubectl delete -n kubernetes-dashboard
  120  kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.6.1/aio/deploy/recommended.yaml
  121  kubectl get all -n kubernetes-dashboard
  122  kubectl get pods
  123  kubectl get secret -n kubernetes-dashboard $(kubectl get serviceaccount kubernetes-dashboard -n kubernetes-dashboard -o jsonpath="{.secrets[0].name}") -o jsonpath="{.data.token}" | base64 --decode
  124  kubectl -n kubernetes-dashboard create token kubernetes-dashboard
  125  kubectl get pods
  126  ls
  127  cd d KubernetesYAMLFile
  128  cd KubernetesYAMLFile
  129  ls
  130  cdDashboard
  131  cd Dashboard
  132  ls
  133  kubectl apply -f dashboard-adminuser.yaml
  134  kubectl -n kubernetes-dashboard create token admin-user
  135  gcloud container clusters get-credentials autopilot-cluster-1 --region us-central1 --project coastal-campus-428405-c4
  136  clear
  137* kubectl de
  138  kubectl get podss
  139  kubectl get pods
  140  git pull
  141  ls
  142  cd KubernetesYAMLFile
  143  git pull
  144  ls
  145  cd Dashboard
  146  ls
  147  history

```
