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

**dowload/upload file từ server về**

 ```bash
# tải file lên host
 scp -r ~/desktop/PUPD/PUBDServerApi root@ip_address:/nguyenhung

# dowload file về
 scp -r root@ip_address:/nguyenhung ~/desktop/PUPD/PUBDServerApi
 ```



**Một số lệnh khác**

 ```bash
pwd // xem thư mục hiện tại
mkdir <folder name> // tạo thư mục mới
rm -r </path>   // xoá thư mục
touch <file name> // Tạo file mới
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


## Phần 2: Tạo SSH Key dể đăng nhập không cần password  

**Cơ chế xác thực bằng SSH Key**

Truy cập vào file config của ssh trên server chỉnh cấu hình như sau



 ```bash
   vi /etc/ssh/sshd_config
 ```

Diều chỉnh 2 dòng config

 ```bash
  PubkeyAuthentication yes
  AuthorizedKeysFile .ssh/authorized_keys

 ```

Tiếp theo là tạo key theo tuần tự lệnh sau

 ```bash

  mkdir keys
  cd keys
  ssh-keygen -t rsa
  id_rsa

 ```

Sau đó xác nhận tạo key bằng cách bấm `Enter` 2 lần.  

 ```bash
   # Kiểm tra đã có file chưa 
   ls -l 
 ```

Nếu đã tồn tại 2 file `id_rsa`  `id_rsa.pub` là đã xong bước tạo key ở máy chủ.

Tiếp theo là coppy file id_rsa.pub` vào `.ssh/authorized_keys` như ở đầu chúng ta đã config file ssh  

 ```bash
  mkdir /root/.ssh
  cp id_rsa.pub /root/.ssh/authorized_keys
 ```

Cấp quyền cho file  

```bash

chmod 600 /root/.ssh/authorized_keys
chmod 700 /root/.ssh
chmod 700 /root
```

**Tạo xác thực ở máy local (MÁC OS)**

```bash

  # Copy từ server về máy mac (nếu có rồi thì sẽ ghi đè lên không sao)
  scp root@34.125.35.106:/keys/id_rsa ~/desktop/

  # Nếu không được thì thêm /root
  scp root@34.125.35.106:/root/keys/id_rsa ~/desktop/
```

Chỉnh sửa cấu hình .SSH ở máy MÁC local

```bash

  # Nếu không có thì tạo
  vim ~/.ssh/config

```

Cấu hình như sau (ý nghĩa của cấu hình này là để phân biệt host nào đi với file nào)
```bash

Host 34.170.198.155 // địa chỉ ip của server centos
 User root
 Port 22
 PreferredAuthentications publickey
 IdentityFile "/Users/nguyenhung/Desktop/id_rsa"  // Nơi chứa file cấu hình tải về

```

Nếu có nhiều server thì clone ra giống vậy.  
cuối cùng kiểm tra thử được chưa.

```bash

# -v để xem log
ssh -v  root@34.125.35.106

```


**Tạo xác thực ở máy local (Window)**

```bash
 scp root@34.125.35.106:/root/keys/id_rsa C:\Users\vanhu\.ssh
```
Không cần cài thêm gì nữa, nếu trường hợp nhiều server thì chưa biết cài.

## Phần 3: Sử dụng Rsync đồng bộ thư mục trên Linux và Windows  

**Sử dụng Rsync đồng bộ thư mục**

 ```bash
# Viết thêm ở đây
 ```

## Phần 4: Sử dụng FUSE để mount ổ đĩa từ xa (server) vào macOS  

**Sử dụng FUSE để mount ổ đĩa từ xa**

 ```bash
# Viết thêm ở đây
 ```

## Phần 5: Sử dụng chương trình sftp - truyền tải file an toàn  

**Sử dụng chương trình sftp - truyền tải file an toàn**

 ```bash
# Viết thêm ở đây
 ```

## Phần 6: crontab chạy script trên máy SSH để backup file định kỳ  

**crontab chạy script trên máy SSH để backup file định kỳ**

 ```bash

 # Mở file crontab
 crontab -e

 # Tạo thư mục log
 touch log.txt

 # Thêm dữ liệu
 * * * * * bash -c 'echo "$(date) - cron is running" >> /root/log.txt'

# Sau khi thêm và lưu thì khởi động lại
sudo systemctl restart crond

 ```

Đối với hệ điều hành Ubuntu thì lưu file bằng cách:  
Nhấn tổ hợp Ctrl + X rồi nhấn phím "Y" để đồng ý

Nếu không dùng được nano thì dùng vim phải gỡ nano  

```bash
  sudo apt update
  sudo apt remove nano
  sudo apt purge nano
  rm -rf ~/.nano
```

Xem cú pháp tại https://crontab.guru/
