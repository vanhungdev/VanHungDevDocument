# Tạo và cài đặt máy ảo VPS Centos để triển khai ứng dụng thực tế
VPS Centos rất ổn định và phù hợp để triển khai ứng dụng thực tế.

## Phần 1: Cấu hình SSH Server Centos 7: 

SSH vào server

 ```bash
SSH root@ip_address
 ```


Sau khi SSH vào centos VPS thì chạy tuần tự một số lênh sau:  
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

Cài vim để chỉnh sửa file

 ```bash
 sudo yum install vim
 ```

Copy file từ server về

 ```bash
 scp -r ~/desktop/PUPD/PUBDServerApi root@ip_address:/nguyenhung  /// Tải file lên host
 scp -r root@ip_address:/nguyenhung ~/desktop/PUPD/PUBDServerApi /// dowload file về
 ```
  
Coppy file từ container docker:

 ```bash

docker cp <container_id>:/usr/share/elasticsearch/config/elasticsearch.yml ~/desktop/docker2  // copy
vim /root/elasticsearch.yml // chú ý thêm /root/

docker cp /root/elasticsearch.yml <container_id>:/usr/share/elasticsearch/config/elasticsearch.yml // ghi đè

// kiểm tra thử có chưa
docker exec -it <container_id> /bin/bash
cat /usr/share/elasticsearch/config/elasticsearch.yml
 ```


Một số lệnh khác

 ```bash
pwd // xem thư mục hiện tại
mkdir // tạo thư mục mới
rm -r /path   // xoá thư mục

 ```

## Phần 2: Cài đặt docker trên Centos 7:

Cài đặt yum

 ```bash
sudo yum install -y yum-utils
 ```

Thêm Docker repo

 ```bash
sudo yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
 ```

Thêm Docker repo

 ```bash
 sudo yum install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
 ```


## Phần 3: Cài đặt docker compose SQL Server, Kafka, Redis, MongoDB, Minio, Portainer:

Tạo file docker Compose có tên docker-compose.yaml như sau:
 ```bash
version: '3'
services:
 mongodb:
   image: mongo:latest
   container_name: mongodb
   restart: always
   ports:
     - "27017:27017"
   environment:
     MONGO_INITDB_ROOT_USERNAME: hungnv165
     MONGO_INITDB_ROOT_PASSWORD: Provanhung77

 minio-server:
   image: minio/minio:latest
   container_name: minio-server
   restart: always
   ports:
     - "9011:9000"
     - "9001:9001"
   environment:
     MINIO_ROOT_USER: hungnv165
     MINIO_ROOT_PASSWORD: Provanhung77
   command: server /data --console-address ":9001"

 portainer:
   image: portainer/portainer-ce
   container_name: portainer
   restart: always
   ports:
     - "9000:9000"
   volumes:
     - /var/run/docker.sock:/var/run/docker.sock
     - portainer_data:/data

 sql-server-container:
   image: mcr.microsoft.com/mssql/server:2017-latest
   container_name: sql-server-container
   restart: always
   ports:
     - "1433:1433"
   environment:
     ACCEPT_EULA: "Y"
     SA_PASSWORD: "Provanhung77"

 redis:
   image: redis:latest
   container_name: redis
   restart: always
   ports:
     - "6379:6379"
   environment:
     REDIS_PASSWORD: Provanhung77
     
 zookeeper:
   container_name: zookeeper
   image: wurstmeister/zookeeper
   ports:
     - "2181:2181"
   networks:
     - kafka-net

 kafka:
   container_name: kafka
   image: wurstmeister/kafka
   ports:
     - "9092:9092"
     - "9093:9093" 
   environment:
     KAFKA_ADVERTISED_LISTENERS: INSIDE://kafka:9093,OUTSIDE://localhost:9092 
     KAFKA_LISTENER_SECURITY_PROTOCOL_MAP: INSIDE:PLAINTEXT,OUTSIDE:PLAINTEXT
     KAFKA_LISTENERS: INSIDE://0.0.0.0:9093,OUTSIDE://0.0.0.0:9092
     KAFKA_INTER_BROKER_LISTENER_NAME: INSIDE
     KAFKA_ZOOKEEPER_CONNECT: zookeeper:2181
   networks:
     - kafka-net

 kafdrop:
   container_name: kafdrop
   image: obsidiandynamics/kafdrop
   ports:
     - "9091:9000"
   environment:
     KAFKA_BROKERCONNECT: kafka:9093
     JVM_OPTS: "-Xms32M -Xmx64M"
   networks:
    - kafka-net
    
volumes:
 portainer_data:
networks:
 kafka-net:
   driver: bridge

 ```

**Chạy Docker Compose:**  
  **Lưu ý:** Để trình chạy file docker compose không gặp lỗi thì chú ý các mục sau:  

1. Chú ý `port` đã bị chiếm trong hệ thống chưa: `2181`, `9092`, `9091`...  
2. Chú ý `container_name` đã có chưa: `zookeeper`, `kafka`, `kafdrop`...  
3. Chú ý `networks` đã có chưa: `kafka-net`.  
4. Chú ý cấu hình các `environment` (biến môi trường) phù hợp.  
5. Chú ý nếu gặp lỗi `The requested image's platform` thì điều chỉnh `image` lại cho phù hợp với platform của bạn.  

Mở terminal và di chuyển đến thư mục chứa tệp docker-compose.yml, sau đó chạy lệnh:  
 ```bash
  docker-compose up -d
 ```

Gỡ chạy lại nếu lỗi:  
 ```bash
  docker-compose down
 ```
 
  Truy cập portainer để kiểm tra các dịch vụ đã chạy đủ chưa:  
 
 ```bash
  http://localhost:9000
 ```


## Phần 4: Cài đặt docker thủ công từng container:

MongoDB:  

 ```bash
    docker run -d --name mongodb \
  -p 27017:27017 \ 
  -e MONGO_INITDB_ROOT_USERNAME=hungnv165 \
  -e MONGO_INITDB_ROOT_PASSWORD=Provanhung77 \
  mongo:latest
 ```

Minio:  

 ```bash
  docker run -d --name minio-server \
  -p 9011:9000 -p 9001:9001 \
  -e MINIO_ROOT_USER=hungnv165 \ 
  -e MINIO_ROOT_PASSWORD=Provanhung77 \
  minio/minio:latest server /data --console-address ":9001"
 ```
Portainer:  

 ```bash
    docker run -d --name portainer \
  -p 9000:9000 \
  -v /var/run/docker.sock:/var/run/docker.sock \
  -v portainer_data:/data \ 
  portainer/portainer-ce

 ```

SQL Server:  

 ```bash
  docker run -d --name sql-server-container 
  -p 1433:1433 
  -e ACCEPT_EULA=Y  
  -e SA_PASSWORD=Provanhung77
  mcr.microsoft.com/mssql/server:2017-latest

 ```
Redis:  

 ```bash
    docker run -d --name redis 
  -p 6379:6379
  -e REDIS_PASSWORD=Provanhung77 
  redis:latest
 ```
Kafka:

1. Tạo networks:  
    ```bash
    docker network create kafka-net
	```	

2. Zookeeper container:
    ```bash
	docker run -d --name zookeeper --network kafka-net -p 2181:2181 wurstmeister/zookeeper
	```	
	2.1. Zookeeper container Đối với macbook M1:
    ```bash
    docker run -d --name zookeeper --network kafka-net -p 2181:2181 arm64v8/zookeeper
    ``` 

3. Kafka container:
    ```bash
   docker run -d --name kafka --network kafka-net -p 9092:9092 -p 9093:9093  --expose 9093  -e KAFKA_ADVERTISED_LISTENERS=INSIDE://kafka:9093,OUTSIDE://localhost:9092  -e KAFKA_LISTENER_SECURITY_PROTOCOL_MAP=INSIDE:PLAINTEXT,OUTSIDE:PLAINTEXT  -e KAFKA_LISTENERS=INSIDE://0.0.0.0:9093,OUTSIDE://0.0.0.0:9092  -e KAFKA_INTER_BROKER_LISTENER_NAME=INSIDE  -e KAFKA_ZOOKEEPER_CONNECT=zookeeper:2181 wurstmeister/kafka
    
    ```
	
4. Kafdrop container:
    ```bash
	docker run -d --name kafdrop --network kafka-net -p 9091:9000 -e KAFKA_BROKERCONNECT=kafka:9093 -e JVM_OPTS="-Xms32M -Xmx64M" obsidiandynamics/kafdrop
    ```
	Kafdrop chưa có cho macbook m1 (arm64v8)
	
