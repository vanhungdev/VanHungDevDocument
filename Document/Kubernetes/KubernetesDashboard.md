# Cài đặt Kubenetes

Tham khảo

```bash
# Cài và khởi tạo Kubeadm Cluster
https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/

# Cách lấy lại token join
https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/

# Cài account
https://github.com/kubernetes/dashboard/blob/master/docs/user/access-control/creating-sample-user.md

# Cài dashboard
https://kubernetes.io/docs/tasks/access-application-cluster/web-ui-dashboard/
```
### Bước 1: Tải k8s 

```bash
curl https://raw.githubusercontent.com/kubernetes/dashboard/v2.0.0-beta6/aio/deploy/recommended.yaml > dashboard-v2-beta6.yaml

```
Mặc định dịch vụ này không truy cập trực tiếp qua IP máy mà phải qua proxy, mở file `dashboard-v2-beta6.yaml`, tìm đến phần kind: `Service` bên trong có tên name: `kubernetes-dashboard`:

```bash
# Mở file
sudo nano dashboard-v2-beta6.yaml
```

Sửa giống như sau
```bash
---

kind: Service
apiVersion: v1
metadata:
  labels:
    k8s-app: kubernetes-dashboard
  name: kubernetes-dashboard
  namespace: kubernetes-dashboard
spec:
  # Đổi kiểu sang NodePort
  type: NodePort
  ports:
    - port: 443
      targetPort: 8443
      # Chọn cổng truy cập qua Node là 3100
      nodePort: 31000
  selector:
    k8s-app: kubernetes-dashboard

---
```
Do cấu hình mặc định của Kubernetes Cluster, cổng được mở ra ngoài phải chọn trong khoảng 30000-32767 sau này bạn có thể sửa cấu hình này với tham số chạy Cluster --service-node-portrange

Xóa Secret có tên kubernetes-dashboard-certs

Đoạn cấu hình này để khởi tạo Secret cấu hình xác thực SSL khi truy cập Dashboard, ta xóa nó đi bằng cách comment như sau:

```bash
# ---

# apiVersion: v1
# kind: Secret
# metadata:
#   labels:
#     k8s-app: kubernetes-dashboard
#   name: kubernetes-dashboard-certs
#   namespace: kubernetes-dashboard
# type: Opaque

```
Sau này sẽ tạo thủ công Secret có tên kubernetes-dashboard-certs thủ công từ certificates sinh ra từ OpenSSL.

### Bước 2 Triển khai dashboard-v2-beta6.yaml

Bây giờ ta sẽ triển khai các thành phần định nghĩa, cấu hình trong dashboard-v2-beta6.yaml, bạn thực hiện lệnh:

```bash
kubectl apply -f dashboard-v2-beta6.yaml
```


### Bước 3 Tạo kubernetes-dashboard-certs, xác thực SSL
Ta sẽ dùng OpenSSL để sinh certificates tự xác thực SSL (ngoài ra khi triển khai thực tế có thể lấy các certificate do mua hoặc miễn phí từ https://letsencrypt.org/ theo Domain), chạy các lệnh sau:



```bash
sudo mkdir certs
sudo chmod 777 -R certs

openssl req -nodes -newkey rsa:2048 -keyout certs/dashboard.key -out certs/dashboard.csr -subj "/C=/ST=/L=/O=/OU=/CN=kubernetes-dashboard"

openssl x509 -req -sha256 -days 365 -in certs/dashboard.csr -signkey certs/dashboard.key -out certs/dashboard.crt

sudo chmod -R 777 certs

```
Sau khi hoàn thành thì các thông tin certificate lưu ở thư mục certs, chạy lệnh sau để tạo Secret

``` bash
kubectl create secret generic kubernetes-dashboard-certs --from-file=certs -n kubernetes-dashboard
```
Giờ truy cập địa chỉ https://<External IP>:31000 sẽ vào màn hình đăng nhập

Nhớ để https

### Cấu hình service account để lấy cấu hình

```bash
sudo nano admin-user.yaml

```
Bỏ code sau vào

```bash

apiVersion: v1
kind: ServiceAccount
metadata:
  name: admin-user
  namespace: kubernetes-dashboard

---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: admin-user
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
- kind: ServiceAccount
  name: admin-user
  namespace: kubernetes-dashboard
---  
apiVersion: v1
kind: Secret
metadata:
  name: admin-user
  namespace: kubernetes-dashboard
  annotations:
    kubernetes.io/service-account.name: "admin-user"   
type: kubernetes.io/service-account-token  
---

```

```bash
kubectl apply -f admin-user.yaml
```

Cách lấy token

```bash
# Nếu không cài service-account-token
kubectl -n kubernetes-dashboard create token admin-user

# Lấy token theo giờ
kubectl -n kubernetes-dashboard create token admin-user --duration=36h

# Nếu cài service-account-token
kubectl -n kubernetes-dashboard describe secret $(kubectl -n kubernetes-dashboard get secret | grep admin-user | awk '{print $1}')

# Hoặc
kubectl get secret admin-user -n kubernetes-dashboard -o jsonpath={".data.token"} | base64 -d


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


## Bước 4 cấu hình Metrics Server

```bash

# Cài đặt metric
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml

# Xem thử
kubectl get pods -n kube-system

# Nếu gặp lỗi TLS
kubectl patch deployment metrics-server -n kube-system --type='json' -p='[{"op": "add", "path": "/spec/template/spec/containers/0/args/-", "value": "--kubelet-insecure-tls"}]'

```
