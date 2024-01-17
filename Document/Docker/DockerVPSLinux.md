# Tạo và cài đặt máy ảo VPS Centos để triển khai ứng dụng thực tế  

Trong phần này sẽ hướng dẫn tất tần tật về docker trên VPS Centos 7.  
Để xem cách cài đặt Centos 7 xem ở phần `SSH Linux Server`


## Phần 1: Cài đặt docker trên Centos 7  

SSH vào server

 ```bash
SSH root@ip_address
 ```

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
   # start docker engine 
   sudo systemctl start docker

   # stop docker engine 
   sudo systemctl stop docker

   # xem trạng thái
   sudo systemctl status docker 
 ```



## Phần 2: Cài đặt docker compose SQL Server, Kafka, Redis, MongoDB, Minio, Portainer  

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

 **Lưu ý biến môi trường Kafka container:**  
 `INSIDE://0.0.0.0:9093,OUTSIDE://0.0.0.0:9092` Chú ý INSIDE OUTSITE khi chạy ở local để kết nối được với Kafdrop
 `KAFKA_ADVERTISED_LISTENERS` Nếu dùng host VPS thì để IP như sau:      
 `KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://34.171.40.194:9092`.   
 Với localhost: `KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://localhost:9092`
 
 **Lưu ý biến môi trường Kafdrop container:**  
 `KAFKA_BROKERCONNECT` Lưu ý nếu dùng VPS thì để IP như sau:  
 `KAFKA_BROKERCONNECT: 34.171.40.194:9092`.



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


## Phần 3: Cài đặt docker thủ công từng container  

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
 docker run -d --name redis -p 6379:6379 -e REDIS_PASSWORD=Provanhung77 redis:latest
 ```


**Elasticsearch Kibana:**    

Tạo Elasticsearch và Kibana container:  

 
 ```bash
# Tạo network
docker network create elastic

# Tạo Elastic Container
docker run --name es01 --net elastic -p 9200:9200 -p 9300:9300 -e "discovery.type=single-node" -e "network.host=0.0.0.0"  -e "xpack.security.enabled=true" -e "ELASTIC_PASSWORD=Provanhung77" -e "xpack.security.http.ssl.enabled=false" -t docker.elastic.co/elasticsearch/elasticsearch:8.11.3

# Tạo kibana container
docker run --name kib01 --net elastic -p 5601:5601 docker.elastic.co/kibana/kibana:8.11.3

```

Việc cài đặt Elasticsearch và Kibana tương đối phức tạp, cần xem thêm phần elasticsearch  


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


## Phần 4: Build image chuyển dữ liệu giữa các server  
**Cách 1: Build image từ container và chuyển file bằng cách SSH vào server đích**


Các bước cài đặt  
 ```bash
	Step 1. Commit container này thành một image mới
	Step 2. Tạo file trên server đích.  
	Step 3. Đẩy image từ server nguồn sang server đích. 
	Step 4. Build lại image trên server đích.
```


Step 1. Commit container này thành một image mới(chạy theo các bước phải comit trước để có data).  

 ```bash
# xem danh sách container đang chạy
docker ps

# tạo một images từ container
docker commit sql-server-container sql-server-container_backup:v.16.01.2024

# login vào docker hub
docker login 
 ```

Step 2. Tạo file trên server đích.  

 ```bash
   # Trên máy chủ đích
   ssh root@34.170.130.200

   # Tạo thư mục /var/lib/docker/tmp nếu chưa có
   mkdir -p /var/lib/docker/tmp
 ```

Step 3. Đẩy image từ server nguồn sang server đích.  


 ```bash

 # lưu lại file gzip và gửi đến server đích
 docker save sql-server-container_backup:v.16.01.2024 | gzip | ssh root@34.170.130.200 'gunzip | docker load'

 # Hoặc có thể tạo thư mục /var/lib/docker/tmp trực tiếp trong dòng lệnh như sau.
 docker save sql-server-container_backup:v.16.01.2024 | gzip | ssh root@34.170.130.200 "mkdir -p /var/lib/docker/tmp && gunzip | docker load"

 ```

Step 4. Build lại image trên server đích.  

 ```bash
 # lưu ý nó dùng password của container cũ
 docker run -d --name sql-server-container -p 1433:1433 sql-server-container_backup:v.16.01.2024
 ```

**Cách 2: Build image từ container và đẩy lên docker hub sau đó build image**

 ```bash
	Step 1. Commit container này thành một image mới.  
	Step 2. Đánh tag cho images.
	Step 3. Push lên docker hub.
	Step 4. Run trên server mới như bình thường.
```


Step 1. Commit container này thành một image mới.  

 ```bash
# Xem danh sách container
docker ps

# Xem danh sách images
docker images

# Commit container thành image
docker commit <container hiện tại> <tên image muốn build>:v.16.01.2024

# Câu lênh mẫu sẽ như sau
docker commit sql-server-container sql-server-container_backup

# login vào docker hub
docker login
 ```

Step 2. Đánh tag cho images.  

 ```bas

# Đánh tag cho images
docker tag sql-server-container_backup vanhungdev/sql-server-container_backup:v.16.01.2024

 ```

Step 3. Push lên docker hub.    

 ```bas
docker push vanhungdev/sql-server-container_backup:v.16.01.2024

 ```


Step 4. Run trên server mới như bình thường.  

 ```bas

# lưu ý nó dùng password của container cũ
docker run -d --name sql-server-container -p 1433:1433 vanhungdev/sql-server-container_backup:v.16.01.2024

 ```
## Phần 5: Backup dữ liệu container về máy local

Cách backup container dữ liệu container về máy local.  

 ```bas

	Step 1. Tạo image backup từ container  
	Step 2. Save image ra file tarball  
	Step 3. Copy file tarball về máy local, ví dụ sử dụng scp  
	Step 4. Load image từ file tarball  
	Step 5. Kiểm tra image đã được tải  
	Step 6. Khởi chạy container từ image backup  

 ```
**Step 1. Tạo image backup từ container**

 ```bas
docker commit sql-server-container sql-server-container_backup:v.16.01.2024

 ```

**Step 2. Save image ra file tarball**

 ```bas
docker save sql-server-container_backup:v.16.01.2024 -o sql-server-backup_16_01_2024.tar

 ```

**Step 3. Copy file tarball về máy local, ví dụ sử dụng scp**

 ```bas
scp root@remote_server:/path/to/sql-server-backup_16_01_2024.tar ~/backups/

 ```

**Step 4. Load image từ file tarball**

 ```bas
docker load -i ~/backups/sql-server-backup_16_01_2024.tar

 ```

**Step 5. Kiểm tra image đã được tải**

 ```bas
docker images

 ```

**Step 6. Khởi chạy container từ image backup**

 ```bas
docker run -d --name sql-server -p 1433:1433 sql-server-container_backup:v.16.01.2024


 ```



## Phần 6: Backup và chuyển tất cả các container cũ

**SQL Server**

Backup SQL Server:  

 ```bas

# Chạy trên server hiện tại
docker commit sql-server-container sql-server-container_backup:v.17.01.2024  
docker save sql-server-container_backup:v.16.01.2024 | gzip | ssh root@34.170.130.200 "mkdir -p /var/lib/docker/tmp && gunzip | docker load"  

#Chạy trên server đích  
docker run -d --name sql-server-container -p 1433:1433 sql-server-container_backup:v.17.01.2024  

```

**Elasticsearch** 

Backup SQL Server:  

 ```bas


#Chạy trên server đích  
docker network create elastic   
 
# Chạy trên server hiện tại
docker commit es01 es01:v.17.01.2024  
docker save es01:v.17.01.2024 | gzip | ssh root@34.170.130.200 "mkdir -p /var/lib/docker/tmp && gunzip | docker load"  

#Chạy trên server đích
docker run --name es01 --net elastic -p 9200:9200 -p 9300:9300  -t es01:v.17.01.2024  


```

**Kibana** 

Backup Kibana Server:  

Đối với kibana không backup kiểu này được nến phải cài lại container  


 ```bas

docker run --name kib01 --net elastic -p 5601:5601 docker.elastic.co/kibana/kibana:8.11.3


# Chạy trên server hiện tại
docker commit kib01 kib01:v.17.01.2024  
docker save kib01:v.17.01.2024 | gzip | ssh root@34.170.130.200 "mkdir -p /var/lib/docker/tmp && gunzip | docker load"  

#Chạy trên server đích  
docker run --name kib01 --net elastic -p 5601:5601 kib01:v.17.01.2024

```
Làm theo hướng dẫn ở phần Elasticsearch  


**Mongodb**  

Backup Mongodb:  

 ```bas

# Chạy trên server hiện tại
docker commit mongodb mongodb:v.17.01.2024  
docker save mongodb:v.17.01.2024 | gzip | ssh root@34.170.130.200 "mkdir -p /var/lib/docker/tmp && gunzip | docker load"  

#Chạy trên server đích  
docker run -d --name mongodb -p 27017:27017  mongodb:v.17.01.2024  
```

**Minio-server** 

Backup Minio-server:  

 ```bas

# Chạy trên server hiện tại
docker commit minio-server minio-server:v.17.01.2024  
docker save minio-server:v.17.01.2024 | gzip | ssh root@34.170.130.200 "mkdir -p /var/lib/docker/tmp && gunzip | docker load"  

#Chạy trên server đích  
 docker run -d --name minio-server -p 9011:9000 -p 9001:9001  minio-server:v.17.01.2024 server /data --console-address ":9001"  
```

**Redis**   

Backup SQL Server:  

 ```bas

# Chạy trên server hiện tại
docker commit redis redis:v.17.01.2024  
docker save redis:v.17.01.2024 | gzip | ssh root@34.170.130.200 "mkdir -p /var/lib/docker/tmp && gunzip | docker load"  
 
#Chạy trên server đích  
docker run -d --name redis -p 6379:6379 minio-server:v.17.01.2024  

```

