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

Truy cập Dashboard bằng service node port
```bash
  # Xem thấy cả service
  k get service -A


  # Kết quả
[root@instance-20240521-043352 ~]# k get service -n kubernetes-dashboard
NAME                        TYPE        CLUSTER-IP      
dashboard-metrics-scraper   ClusterIP   10.109.109.3
kubernetes-dashboard        ClusterIP    10.101.16.230
```

Bây giờ chúng ta cần chỉnh Service có tên kubernetes-dashboard từ `ClusterIP` thành `NodePort` là có thể truy cập được.

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

Bây giờ chung ta fw port 
```bash
sudo iptables -t nat -A PREROUTING -p tcp --dport 32507 -j DNAT --to-destination <Minikube IP>:32507

```

 Thực hiện chức năng DNAT (Destination Network Address Translation) trong iptables, một công cụ tường lửa phổ biến trên Linux. Cụ thể, lệnh này sẽ chuyển hướng lưu lượng truy cập TCP đến trên cổng 32507 của máy chủ (VPS) của bạn đến cổng 32507 của một máy chủ khác có địa chỉ IP là 192.168.49.2.


Như vậy là từ IP của VPS chúng ta có thể truy cập được Dashboard của minikube.

```bash
http://<VPS IP>:32507

```
