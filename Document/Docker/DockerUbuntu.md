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

## Cách 2: Cài đặt docker trên Ubuntu theo hướng dẫn trên trang chủ
hoặc làm theo trang chủ theo link sau:  
https://docs.docker.com/engine/install/ubuntu/

gỡ bỏ docker phiên bản cũ:

 ```bash
 for pkg in docker.io docker-doc docker-compose docker-compose-v2 podman-docker containerd runc; do sudo apt-get remove $pkg; done
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
  sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin -y
```


**Thông tin thêm**

Start và kiểm tra trạng thái docker

 ```bash
   # start docker engine 
   sudo service docker start

   # xem trạng thái 
   sudo service docker status

   # stop docker engine
   sudo service docker stop 
 ```


Ubuntu khác với centos ở cách cài package và khởi động các dịch vụ  

 ```bash

  # khởi động dịch vụ
  sudo service <service>

  # cập nhật package
  apt update

  # cài package
  apt install <tên package>
 ```


```bash
 # Đối với docker mới
 docker compose up

 # docker cũ
 docker-compose up
```


Portainer trên Ubuntu

```bash

sudo docker run -d --restart=always -p 8000:8000 -p 9000:9000 --name portainer --volume /var/run/docker.sock:/var/run/docker.sock -v /opt/portainer/data:/data portainer/portainer-ce:latest


```


```bash
    7  sudo apt install net-tools
    8  sudo apt install vim
    9  clear
   10  sudo apt install gedit
   11  sudo apt install nautilus
   12  sudo apt install open-vm-tools

```
