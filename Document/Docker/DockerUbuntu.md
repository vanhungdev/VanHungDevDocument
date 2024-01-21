# Tạo và cài đặt máy ảo VPS Ubuntu để triển khai ứng dụng thực tế  

Trong phần này sẽ hướng dẫn tất tần tật về docker trên VPS Ubuntu.  

## Phần 1: Cài đặt docker trên Ubuntu 

SSH vào server

 ```bash
SSH root@ip_address
 ```

Update package

 ```bash
   sudo apt update
 ```

Cài đặt docker

 ```bash
  sudo apt install docker.io
 ```

Cài dặt docker và docker compose

 ```bash

   sudo curl -L "https://github.com/docker/compose/releases/download/1.25.0/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose

   # Cấp quyền
   sudo chmod +x /usr/local/bin/docker-compose
 ```

Start và kiểm tra trạng thái docker

 ```bash
   # start docker engine 
   sudo service docker start

   # xem trạng thái 
   sudo service docker status

   # stop docker engine
   sudo service docker stop 
 ```
## Cách 2: Cài đặt docker trên Ubuntu theo hướng dẫn trên trang chủ
hoặc làm theo trang chủ theo link sau:  
https://docs.docker.com/engine/install/ubuntu/

gỡ bỏ docker phiên bản cũ:

 ```bash
SSH root@ip_address
 ```

Set up Docker's apt repository:  

Chạy lần lượt tuần dòng lệnh.  

 ```bash
  # Add Docker's official GPG key:
  sudo apt-get update
  sudo apt-get install ca-certificates curl gnupg
  sudo install -m 0755 -d /etc/apt/keyrings
  curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
  sudo chmod a+r /etc/apt/keyrings/docker.gpg
  
  # Add the repository to Apt sources:
  echo \
    "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
    $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
    sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
  sudo apt-get update
 ```

Cài đặt docker compose

```bash
  sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```


**Thông tin thêm**

Ubuntu khác với centos ở cách cài package và khởi động các dịch vụ  

 ```bash

  # khởi động dịch vụ
  sudo service <service>

  # cập nhật package
  apt update

  # cài package
  apt install <tên package>
 ```
