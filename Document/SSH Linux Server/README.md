# Tạo và cài đặt máy ảo VPS Centos để triển khai ứng dụng thực tế
VPS Centos rất ổn định và phù hợp để triển khai ứng dụng thực tế.

## Phần 1: Cấu hình SSH Server Centos 7: 

**SSH vào server**

 ```bash
SSH root@ip_address
 ```


**Lênh tuần tự để setup server centos**
 ```bash

whoami
cat /etc/os-release
sudo passwd
su
systemctl stop sshd
yum remove openssh-server
yum install openssh openssh-server openssh-clients openssl-libs -y
systemctl start sshd
systemctl enable sshd
firewall-cmd --add-port=22/tcp --permanent
firewall-cmd --reload
systemctl restart sshd
systemctl status sshd
 ```

**Cài vim để chỉnh sửa file**

 ```bash
 sudo yum install vim
 ```

Copy file từ server về

 ```bash
#  Tải file lên host
 scp -r ~/desktop/PUPD/PUBDServerApi root@ip_address:/nguyenhung

#dowload file về
 scp -r root@ip_address:/nguyenhung ~/desktop/PUPD/PUBDServerApi
 ```



**Một số lệnh khác**

 ```bash
pwd // xem thư mục hiện tại
mkdir // tạo thư mục mới
rm -r /path   // xoá thư mục

 ```

## Phần 2: Cài đặt docker trên Centos 7  

**Cài đặt yum**

 ```bash
  sudo yum install -y yum-utils
 ```

**Thêm Docker repo**

 ```bash
  sudo yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
 ```

**Cài dặt docker và docker compose**

 ```bash
  sudo yum install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
 ```

**Start và kiểm tra trạng thái docker**

 ```bash
   # start docker engine 
   sudo systemctl start docker

   # stop docker engine 
   sudo systemctl stop docker

   # xem trạng thái
   sudo systemctl status docker 
 ```

**Coppy file từ container docker:**

 ```bash

# Lênh coppy file
docker cp <container_id>:/usr/share/elasticsearch/config/elasticsearch.yml ~/desktop/docker2

# Lệnh chỉnh file (chú ý thêm /root/)
vim /root/elasticsearch.yml 

docker cp /root/elasticsearch.yml <container_id>:/usr/share/elasticsearch/config/elasticsearch.yml // ghi đè

# kiểm tra thử có chưa (exec vào môi trường container)
docker exec -it <container_id> /bin/bash

cat /usr/share/elasticsearch/config/elasticsearch.yml

 ```
