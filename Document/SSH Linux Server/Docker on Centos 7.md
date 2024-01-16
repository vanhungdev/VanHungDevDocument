# Tạo và cài đặt máy ảo VPS Centos để triển khai ứng dụng thực tế  

Trong phần này sẽ hướng dẫn tất tần tật về docker trên VPS Centos 7.

## Phần 1: Cài đặt docker trên Centos 7:

Cài đặt yum

 ```bash
sudo yum install -y yum-utils
 ```

Thêm Docker repo

 ```bash
sudo yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
 ```

Cài dặt docker và docker compose

 ```bash
 sudo yum install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
 ```

Start và kiểm tra trạng thái docker

 ```bash

   sudo systemctl start docker // start docker engine 
   sudo systemctl stop docker // stop docker engine 
   sudo systemctl status docker // xem trạng thái
 ```



## Phần 2: Cài đặt docker compose SQL Server, Kafka, Redis, MongoDB, Minio, Portainer:

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


## Phần 3: Cài đặt docker thủ công từng container:

**Portainer:**  

 ```bash
 docker run -d --name portainer -p 9000:9000 -v /var/run/docker.sock:/var/run/docker.sock -v portainer_data:/data portainer/portainer-ce

 ```

**MongoDB:**  

 ```bash
 docker run -d --name mongodb -p 27017:27017 -e MONGO_INITDB_ROOT_USERNAME=hungnv165 -e MONGO_INITDB_ROOT_PASSWORD=Provanhung77 mongo:latest
 ```

**Minio:**  

 ```bash
  docker run -d --name minio-server -p 9011:9000 -p 9001:9001 -e MINIO_ROOT_USER=hungnv165 -e MINIO_ROOT_PASSWORD=Provanhung77 minio/minio:latest server /data --console-address ":9001"
 ```

**SQL Server:**   

 ```bash
  docker run -d --name sql-server-container -p 1433:1433 -e ACCEPT_EULA=Y  -e SA_PASSWORD=Provanhung77 mcr.microsoft.com/mssql/server:2017-latest

 ```
**Redis:**    

 ```bash
 docker run -d --name redis -p 6379:6379-e REDIS_PASSWORD=Provanhung77 redis:latest
 ```
**Kafka:**  

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


## Phần 4: Build image chuyển dữ liệu giữa các server: 
**Cách 1: Build image từ container và chuyển file bằng cách SSH vào server đích**
1. Commit container này thành một image mới.  

 ```bash
docker ps

docker commit sql-server-container sql-server-container_backup

docker login // login vào docker hub
 ```

2. Tạo file trên server đích.  

 ```bash
   # Trên máy chủ đích
   ssh root@34.125.19.138

   # Tạo thư mục /var/lib/docker/tmp nếu chưa có
   mkdir -p /var/lib/docker/tmp
 ```

3. Đẩy image từ server nguồn sang server đích.  

 ```bash
 docker save sql-server-container_backup | gzip | ssh root@34.125.19.138 'gunzip | docker load'

 # Hoặc có thể tạo thư mục /var/lib/docker/tmp trực tiếp trong dòng lệnh như sau.
 docker save sql-server-container_backup | gzip | ssh root@34.125.19.138 "mkdir -p /var/lib/docker/tmp && gunzip | docker load"

 ```

4. Build lại image trên server đích.  

 ```bash
 docker run -d --name sql-server-container -p 1433:1433 -e ACCEPT_EULA=Y  -e SA_PASSWORD=Provanhung77 sql-server-container_backup
 ```

**Cách 2: Build image từ container và đẩy lên docker hub sau đó build image**

1. Commit container này thành một image mới.  

 ```bash
# Xem danh sách container
docker ps

#Xem danh sách images
docker images

#Commit container thành image
docker commit <container hiện tại> <tên image muốn build>:v.16.01.2024

# Câu lênh mẫu sẽ như sau
docker commit sql-server-container sql-server-container_backup

# login vào docker hub
docker login
 ```

2. Đánh tag cho image.  

 ```bas
docker tag sql-server-container_backup vanhungdev/sql-server-container_backup:v.16.01.2024

 ```

3. push lên docker hub.    

 ```bas
docker push vanhungdev/sql-server-container_backup:v.16.01.2024

 ```


4. Run trên server mới như bình thường.  

 ```bas

docker run -d --name sql-server-container -p 1433:1433 -e ACCEPT_EULA=Y  -e SA_PASSWORD=Provanhung77 vanhungdev/sql-server-container_backup:v.16.01.2024

 ```

	
