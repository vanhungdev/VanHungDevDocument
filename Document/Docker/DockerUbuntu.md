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
