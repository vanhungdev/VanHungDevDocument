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

   # Bật docker chạy cùng hệ thống
   sudo systemctl enable docker

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
 docker run -d --name portainer --restart=always -p 9000:9000 -v /var/run/docker.sock:/var/run/docker.sock -v portainer_data:/data portainer/portainer-ce

# Nếu quên pass
docker volume rm portainer_data
 ```

**MongoDB:**  

 ```bash
 docker run -d --name mongodb --restart=always -p 27017:27017 -e MONGO_INITDB_ROOT_USERNAME=hungnv165 -e MONGO_INITDB_ROOT_PASSWORD=Provanhung77 mongo:latest
 ```

**Minio:**  

 ```bash
  docker run -d --name minio-server --restart=always -p 9011:9000 -p 9001:9001 -e MINIO_ROOT_USER=hungnv165 -e MINIO_ROOT_PASSWORD=Provanhung77 minio/minio:latest server /data --console-address ":9001"

# Hoặc

version: "3"
services:
  minio:
    image: minio/minio
    container_name: minio
    ports:
      - "9111:9000"
      - "9222:9001"
    volumes:
      - ./storage:/data
    environment:
      MINIO_ROOT_USER: hungnv165
      MINIO_ROOT_PASSWORD: Provanhung77
    command: server --console-address ":9001" /data


version: "3"
services:
  minio:
    image: minio/minio:RELEASE.2022-11-11T03-44-20Z
    container_name: minio
    ports:
      - 9111:9000
      - 9222:9001
    volumes:
      - ./minio_storage:/data
    command: server --console-address ":9001" /data
    environment:
      MINIO_ROOT_USER: hungnv165
      MINIO_ROOT_PASSWORD: Provanhung77
networks: {}


# public Image hoặc update bằng server thì dùng host này
http://34.170.212.251:9111/bloghung/mobisalevn/dev/images/posts/wepik-fox-coder-logo-20240528151349HHQz (1).jpeg

# Console port (dùng để vào console dashboard)
http://34.170.212.251:9222
 ```

**SQL Server:**   

 ```bash
  docker run -d --name sql-server-container --restart=always -p 1433:1433 -e ACCEPT_EULA=Y  -e SA_PASSWORD=Provanhung77 mcr.microsoft.com/mssql/server:2017-latest

 ```

**Redis:**    

 ```bash
 docker run -d --name redis --restart=always -p 6379:6379 -e REDIS_PASSWORD=Provanhung77 redis:latest
 ```

**Centos:**    

 ```bash

docker run -dit --name centos7 -v /data:/data centos:7
yum update -y
yum install -y vim wget net-tools
yum install -y yum-utils

 ```

**Công cụ quản lý docker compose - Dockge:**    

Chạy bằng lệnh

```bash

sudo docker run -d --name dockge \
  --restart always \
  -p 5001:5001 \
  -v dockge-data:/app/data \
  -v dockge-stacks:/opt/stacks \
  -v /var/run/docker.sock:/var/run/docker.sock \
  -e DOCKGE_STACKS_DIR=/opt/stacks \
  louislam/dockge:1


```

docker compose file  

 ```bash

	# Tạo thư mục lưu trữ ngăn xếp của bạn và lưu trữ ngăn xếp của Dockge
	mkdir -p /opt/stacks /opt/dockge
	cd /opt/dockge
	
	# Download the compose.yaml file
	curl https://raw.githubusercontent.com/louislam/dockge/master/compose.yaml --output compose.yaml
	
	# Chạy lệnh docker compose để cài Dockge
	docker compose up -d
	
	# Nếu dùng docker phiên bản cũ thì chạy lệnh
	# docker-compose up -d



 ```

Docker file  

 ```bash

	version: "3.8"
	services:
	  dockge:
	    image: louislam/dockge:1
	    restart: unless-stopped
	    ports:
	      # Host Port : Container Port
	      - 5001:5001
	    volumes:
	      - /var/run/docker.sock:/var/run/docker.sock
	      - ./data:/app/data
	        
	      # If you want to use private registries, you need to share the auth file with Dockge:
	      # - /root/.docker/:/root/.docker
	
	      # Stacks Directory
	      # ⚠️ READ IT CAREFULLY. If you did it wrong, your data could end up writing into a WRONG PATH.
	      # ⚠️ 1. FULL path only. No relative path (MUST)
	      # ⚠️ 2. Left Stacks Path === Right Stacks Path (MUST)
	      - /opt/stacks:/opt/stacks
	    environment:
	      # Tell Dockge where is your stacks directory
	      - DOCKGE_STACKS_DIR=/opt/stacks
```


**Elasticsearch Kibana:**    

Tạo Elasticsearch và Kibana container:  

 
 ```bash
# Tạo network
docker network create elastic

# Tạo Elastic Container
docker run --name es01 --restart=always --net elastic -p 9200:9200 -p 9300:9300 -e "discovery.type=single-node" -e "network.host=0.0.0.0"  -e "xpack.security.enabled=true" -e "ELASTIC_PASSWORD=Provanhung77" -e "xpack.security.http.ssl.enabled=false" -t docker.elastic.co/elasticsearch/elasticsearch:8.11.3

# Tạo kibana container
docker run --name kib01 --restart=always --net elastic -p 5601:5601 docker.elastic.co/kibana/kibana:8.11.3

```

Việc cài đặt Elasticsearch và Kibana tương đối phức tạp, cần xem thêm phần elasticsearch  


**Kafka:**  

1. Tạo networks:  
    ```bash
    docker network create kafka-net
	```	

2. Zookeeper container:
    ```bash
	docker run -d --name --restart=always zookeeper --network kafka-net -p 2181:2181 wurstmeister/zookeeper
	```	
	2.1. Zookeeper container Đối với macbook M1:
    ```bash
    docker run -d --name --restart=always zookeeper --network kafka-net -p 2181:2181 arm64v8/zookeeper
    ``` 

3. Kafka container:
    ```bash
   docker run -d --name kafka --restart=always --network kafka-net -p 9092:9092 -p 9093:9093  --expose 9093  -e KAFKA_ADVERTISED_LISTENERS=INSIDE://kafka:9093,OUTSIDE://localhost:9092  -e KAFKA_LISTENER_SECURITY_PROTOCOL_MAP=INSIDE:PLAINTEXT,OUTSIDE:PLAINTEXT  -e KAFKA_LISTENERS=INSIDE://0.0.0.0:9093,OUTSIDE://0.0.0.0:9092  -e KAFKA_INTER_BROKER_LISTENER_NAME=INSIDE  -e KAFKA_ZOOKEEPER_CONNECT=zookeeper:2181 wurstmeister/kafka
    
    ```
	
4. Kafdrop container:
    ```bash
	docker run -d --name kafdrop --network --restart=always kafka-net -p 9091:9000 -e KAFKA_BROKERCONNECT=kafka:9093 -e JVM_OPTS="-Xms32M -Xmx64M" obsidiandynamics/kafdrop
    ```
	Kafdrop chưa có cho macbook m1 (arm64v8)

Redis new
```bash
services:
  redis:
    image: redis:alpine
    container_name: redis
    restart: unless-stopped
    ports:
      - "6379:6379"
    volumes:
      - redis-data:/data
    environment:
      - INSIGHT_WEB_PORT=8001
    networks:
      - redis-network
    command: ["redis-server", "--appendonly", "yes"]
    healthcheck:
      test:
          - CMD
          - redis-cli
          - ping
      retries: 3
      timeout: 5s

  redisinsight:
    image: redis/redisinsight:latest
    container_name: redisinsight
    restart: unless-stopped
    environment:
      - REDIS_HOST=redis
      - REDIS_PORT=6379
      - REDISINSIGHT_HOST=0.0.0.0
      - REDISINSIGHT_ACCEPT_LICENSE=true  # Accept EULA and Privacy Settings
    ports:
      - "5540:5540"
    volumes:
      - redisinsight-data:/db
    networks:
      - redis-network
    depends_on:
      - redis

volumes:
  redis-data:
    driver: local
  redisinsight-data:
    driver: local

networks:
  redis-network:
    driver: bridge
```



