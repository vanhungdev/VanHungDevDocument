# Tạo và cài đặt máy ảo nginx để triển khai ứng dụng thực tế
Viết cái gì đó ở đây

## Phần 1: Cấu hình SSH Server Centos 7: 

SSH vào server

whoami
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

## Phần 2: Cài đặt docker trên Centos 7:
  **Lưu ý:** Viết cái gì đó ở đây:  

## Phần 3: Cài đặt docker compose SQL Server, Kafka, Redis, MongoDB, minio, portainer:

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
 
  Truy cập portainer để kiểm tra các dịch vụ đã chạy đủ chưa:  
 
 ```bash
  http://localhost:9000
 ```


