# Cấu hình MiniKube và systemctl
- 
systemctl
```bash
# Download the latest release with the command
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
```

Cấp quyền và kiểm tra version
```bash
   chmod +x kubectl
   18  sudo mv ./kubectl /usr/local/bin/kubectl
   19  kubectl version
```


MiniKube
```bash
grep -E --color 'vmx|svm' /proc/cpuinfo

curl -Lo minikube https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64   && chmod +x minikube

  sudo mkdir -p /usr/local/bin/
  sudo install minikube /usr/local/bin/

  minikube start --driver=docker --force


  minikube status

  kubectl get nodes

  # Cài đặt Xem CPU, RAM
  minikube addons enable metrics-server
   
  minikube dashboard

  kubectl proxy --address=0.0.0.0 --accept-hosts='.*'
```

```bash
#Truy cập dashbload


http://34.29.18.204:8001/api/v1/namespaces/kubernetes-dashboard/services/http:kubernetes-dashboard:/proxy/#/about?namespace=default

```
