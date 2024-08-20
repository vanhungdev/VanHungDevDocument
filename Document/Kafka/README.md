# Kafka Master

## Phần 1: Gới thiệu chung
Viết 1 cái gì đó ở đây 

**Thông tin Broker Kafka đã có sẵn trên VPS có thể sử dụng.**   
**Lưu ý:** ad


2. Kafdrop:  
    ```bash
    34.125.35.106:9091
	```	
	
3. Portainer:  
    ```bash
    34.125.35.106:9000/
	
	Username: admin
	Password: Provanhung77
	```	

3. BootstrapServers (mạng công ty không truy cập được, mạng thường và VPN thì có thể):  
    ```bash
    34.125.35.106:9092
	```	
	
## Phần 2: Cài đặt Kafka bằng docker compose:  

**Để sử dụng được Kafka cần có 3 container cần thiết sau:**   

1. **Zookeeper** - Quản lý Kafka
2. **Kafka** - Kafka broker
3. **Kafdrop** - Công cụ theo dõi và quản lý cần thiết cho việc load test  

Tạo file docker Compose có tên `docker-compose.yaml` như sau:  


```bash
version: "3"
services:
  zookeeper:
    container_name: zookeeper
    image: wurstmeister/zookeeper
    ports:
      - 2181:2181
    networks:
      - kafka-net
  kafka:
    container_name: kafka
    image: wurstmeister/kafka
    ports:
      - 9092:9092
      - 9093:9093
      - 29092:29092
      - 9999:9999
    environment:
      KAFKA_ADVERTISED_LISTENERS: INSIDE://kafka:9093,OUTSIDE://  <your-ip>    :9092,DOCKER://host.docker.internal:29092
      KAFKA_LISTENER_SECURITY_PROTOCOL_MAP: INSIDE:PLAINTEXT,OUTSIDE:PLAINTEXT,DOCKER:PLAINTEXT
      KAFKA_LISTENERS: INSIDE://0.0.0.0:9093,OUTSIDE://0.0.0.0:9092,DOCKER://0.0.0.0:29092
      KAFKA_INTER_BROKER_LISTENER_NAME: INSIDE
      KAFKA_ZOOKEEPER_CONNECT: zookeeper:2181
      KAFKA_BROKER_ID: 1
      KAFKA_LOG4J_LOGGERS: kafka.controller=INFO,kafka.producer.async.DefaultEventHandler=INFO,state.change.logger=INFO
      KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR: 1
      KAFKA_TRANSACTION_STATE_LOG_REPLICATION_FACTOR: 1
      KAFKA_TRANSACTION_STATE_LOG_MIN_ISR: 1
      KAFKA_JMX_PORT: 9999
      KAFKA_JMX_HOSTNAME: ${DOCKER_HOST_IP:-127.0.0.1}
      KAFKA_AUTHORIZER_CLASS_NAME: kafka.security.authorizer.AclAuthorizer
      KAFKA_ALLOW_EVERYONE_IF_NO_ACL_FOUND: "true"
    networks:
      - kafka-net
  kafdrop:
    container_name: kafdrop
    image: obsidiandynamics/kafdrop
    ports:
      - 9091:9000
    environment:
      KAFKA_BROKERCONNECT: kafka:9093
      JVM_OPTS: -Xms32M -Xmx64M
    networks:
      - kafka-net
    expose:
      - "9093" # Expose Kafka listener port to other containers
networks:
  kafka-net:
    driver: bridge


```

 **Chạy Docker Compose:**  
  **Lưu ý:** Để trình chạy file docker compose không gặp lỗi thì chú ý các mục sau:  

1. Chú ý `port` đã bị chiếm trong hệ thống chưa: `2181`, `9092`, `9091`.  
2. Chú ý `container_name` đã có chưa: `zookeeper`, `kafka`, `kafdrop`.  
3. Chú ý `networks` đã có chưa: `kafka-net`.  
4. Chú ý cấu hình các `environment` (biến môi trường) phù hợp.  
5. Chú ý nếu gặp lỗi `The requested image's platform` thì điều chỉnh `image` lại cho phù hợp với platform của bạn.  

Mở terminal và di chuyển đến thư mục chứa tệp docker-compose.yml, sau đó chạy lệnh:  
 ```bash
  docker-compose up -d
 ```
 
  Truy cập Kafdrop:  
 
 ```bash
  http://localhost:9091
 ```
   
 Nếu chưa biết tạo file docker compose trên linux/centos thì làm như sau, ở window chỉ cần cd đến thư mục chứa file và chạy:

1. Cài docker-compose nếu chưa có (chạy từng dòng):  
    ```bash
     docker-compose --version
     sudo curl -L "https://github.com/docker/compose/releases/latest/download/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
     sudo chmod +x /usr/local/bin/docker-compose
    ``` 

2. Cài vim nếu chưa  (chạy từng dòng):  
    ```bash
    sudo yum install vim
    vim --version
    ``` 

3. Tạo thư mục lưu file dockerconpose (chạy từng dòng):  
    ```bash
   ls
   mkdir kafka-docker-compose-file
   cd kafka-docker-compose-file
   vim docker-compose.yaml
    ``` 
**Cách xử lý file với Vim:**  
   Sa khi cấu hình xong file docker compose xong thì nhấn: `esc + :wq` để lưu và thoát.  
   Phím `i` để chỉnh sửa.  
   Phím `gg` để đưa con trỏ chuột về đầu file.   
   Phím `ggdG` để xoá toàn bộ file.  
   Phím `dG` để xoá từ vị trí con trỏ chuột về cuối file và lưu ý `G viết hoa`.  


**Lưu lý các biến môi trường sau:**   

1. **Kafka container:**  
 `INSIDE://0.0.0.0:9093,OUTSIDE://0.0.0.0:9092` Chú ý INSIDE OUTSITE khi chạy ở local để kết nối được với Kafdrop
 `KAFKA_ADVERTISED_LISTENERS` Nếu dùng host VPS thì để IP như sau:      
 `KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://34.171.40.194:9092`.   
 Với localhost: `KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://localhost:9092`
 
2. **Kafdrop container:**  
 `KAFKA_BROKERCONNECT` Lưu ý nếu dùng VPS thì để IP như sau:  
 `KAFKA_BROKERCONNECT: 34.171.40.194:9092`. 
 Với localhost thì `KAFKA_BROKERCONNECT: localhost:9092`.

 
## Phần 3: Cài đặt Kafka bằng docker run từng container (không muốn chạy compose):

0. Tạo networks:  
    ```bash
    docker network create kafka-net
	```	

1. Zookeeper container:
    ```bash
	docker run -d --name  zookeeper --restart=always --network kafka-net -p 2181:2181 wurstmeister/zookeeper
	```	
	1.1. Zookeeper container Đối với macbook M1:
    ```bash
    docker run -d --name zookeeper --restart=always --network kafka-net -p 2181:2181 arm64v8/zookeeper
    ``` 

1. Kafka container:
    ```bash
   docker run -d --name kafka --restart=always --network kafka-net -p 9092:9092 -p 9093:9093  --expose 9093  -e KAFKA_ADVERTISED_LISTENERS=INSIDE://kafka:9093,OUTSIDE://localhost:9092  -e KAFKA_LISTENER_SECURITY_PROTOCOL_MAP=INSIDE:PLAINTEXT,OUTSIDE:PLAINTEXT  -e KAFKA_LISTENERS=INSIDE://0.0.0.0:9093,OUTSIDE://0.0.0.0:9092  -e KAFKA_INTER_BROKER_LISTENER_NAME=INSIDE  -e KAFKA_ZOOKEEPER_CONNECT=zookeeper:2181 wurstmeister/kafka
    
    ```
	
1. Kafdrop container:
    ```bash
	docker run -d --name kafdrop --restart=always --network kafka-net -p 9091:9000 -e KAFKA_BROKERCONNECT=kafka:9093 -e JVM_OPTS="-Xms32M -Xmx64M" obsidiandynamics/kafdrop
    ```
	Kafdrop chưa có cho macbook m1 (arm64v8)
	
## Phần 5: Truy cập và quản lý topic Kafka:  
 
 Để sử dụng được kafka cũng ta cần kiểm tra xem container đã run chưa và cấu hình thêm topic cho nó. Cấu hình các partitions để xử linh hoạt hơn.
 Nếu không rành gõ lệnh các bạn có thể cấu hình trực tiếp bằng `Kafdrop` 

1. Exec vào contaier:  
    ```bash
    docker exec -it kafka /bin/bash
	```	

2. Tạo topic:  
    ```bash
    kafka-topics.sh --create --topic topic-events1 --bootstrap-server localhost:9092
	```	
	
3. Tạo topic và partition:  
    ```bash
    kafka-topics.sh --create --topic topic-events2 --partitions 3 --replication-factor 1 --bootstrap-server localhost:9092
	```	
	
4. Tăng số lượng partitions:  
    ```bash
    kafka-topics.sh --bootstrap-server localhost:9092 --topic topic-events2 --alter --partitions 5
	```		

5. Xem mô tả topic:  
    ```bash
    kafka-topics.sh --bootstrap-server localhost:9092 --topic topic-events2 --describe
	```	
6. Producer:  
    ```bash
    kafka-console-producer.sh --topic topic-events2 --bootstrap-server localhost:9092
	```		

7. Consumer:  
    ```bash
    kafka-console-consumer.sh --topic topic-events2 --from-beginning --bootstrap-server localhost:9092
	```	
	
8. Consumer với group:  
    ```bash
    kafka-console-consumer.sh  --bootstrap-server localhost:9092 --topic topic-events2 --group group-topic-events2-001

    ``` 
    Việc muốn đọc toàn bộ message chỉ cần thêm `--from-beginning`

9. Xem thông tin chi tiết consumer group:  
    ```bash
    kafka-consumer-groups.sh --bootstrap-server localhost:9092 --describe --group group-topic-events2-001

    ``` 


## KAFKA Connect

```bash
version: "3"
services:
  zookeeper:
    container_name: zookeeper
    image: wurstmeister/zookeeper
    ports:
      - 2181:2181
    networks:
      - kafka-net
  kafka:
    container_name: kafka
    image: wurstmeister/kafka
    ports:
      - 9092:9092
      - 9093:9093
      - 29092:29092
      - 9999:9999
    environment:
      KAFKA_ADVERTISED_LISTENERS: INSIDE://kafka:9093,OUTSIDE://34.135.32.57:9092,DOCKER://host.docker.internal:29092
      KAFKA_LISTENER_SECURITY_PROTOCOL_MAP: INSIDE:PLAINTEXT,OUTSIDE:PLAINTEXT,DOCKER:PLAINTEXT
      KAFKA_LISTENERS: INSIDE://0.0.0.0:9093,OUTSIDE://0.0.0.0:9092,DOCKER://0.0.0.0:29092
      KAFKA_INTER_BROKER_LISTENER_NAME: INSIDE
      KAFKA_ZOOKEEPER_CONNECT: zookeeper:2181
      KAFKA_BROKER_ID: 1
      KAFKA_LOG4J_LOGGERS: kafka.controller=INFO,kafka.producer.async.DefaultEventHandler=INFO,state.change.logger=INFO
      KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR: 1
      KAFKA_TRANSACTION_STATE_LOG_REPLICATION_FACTOR: 1
      KAFKA_TRANSACTION_STATE_LOG_MIN_ISR: 1
      KAFKA_JMX_PORT: 9999
      KAFKA_JMX_HOSTNAME: ${DOCKER_HOST_IP:-127.0.0.1}
      KAFKA_AUTHORIZER_CLASS_NAME: kafka.security.authorizer.AclAuthorizer
      KAFKA_ALLOW_EVERYONE_IF_NO_ACL_FOUND: "true"
    networks:
      - kafka-net
  kafdrop:
    container_name: kafdrop
    image: obsidiandynamics/kafdrop
    ports:
      - 9091:9000
    environment:
      KAFKA_BROKERCONNECT: kafka:9093
      JVM_OPTS: -Xms32M -Xmx64M
    networks:
      - kafka-net
    expose:
      - "9093" # Expose Kafka listener port to other containers
  schema-registry:
    container_name: schema-registry
    image: confluentinc/cp-schema-registry:6.2.0
    ports:
      - 8081:8081
    environment:
      SCHEMA_REGISTRY_HOST_NAME: schema-registry
      SCHEMA_REGISTRY_KAFKASTORE_BOOTSTRAP_SERVERS: kafka:9093
  kafka-connect:
    container_name: kafka-connect
    image: confluentinc/cp-kafka-connect:latest
    ports:
      - 8083:8083
    environment:
      CONNECT_BOOTSTRAP_SERVERS: kafka:9093
      CONNECT_REST_ADVERTISED_HOST_NAME: connect
      CONNECT_REST_PORT: 8083
      CONNECT_GROUP_ID: compose-connect-group
      CONNECT_CONFIG_STORAGE_TOPIC: docker-connect-configs
      CONNECT_CONFIG_STORAGE_REPLICATION_FACTOR: 1
      CONNECT_OFFSET_STORAGE_TOPIC: docker-connect-offsets
      CONNECT_OFFSET_STORAGE_REPLICATION_FACTOR: 1
      CONNECT_STATUS_STORAGE_TOPIC: docker-connect-status
      CONNECT_STATUS_STORAGE_REPLICATION_FACTOR: 1
      CONNECT_KEY_CONVERTER: io.confluent.connect.avro.AvroConverter
      CONNECT_KEY_CONVERTER_SCHEMA_REGISTRY_URL: http://schema_registry:8081
      CONNECT_VALUE_CONVERTER: io.confluent.connect.avro.AvroConverter
      CONNECT_VALUE_CONVERTER_SCHEMA_REGISTRY_URL: http://schema_registry:8081
      CONNECT_INTERNAL_KEY_CONVERTER: org.apache.kafka.connect.json.JsonConverter
      CONNECT_INTERNAL_VALUE_CONVERTER: org.apache.kafka.connect.json.JsonConverter
      CONNECT_ZOOKEEPER_CONNECT: zookeeper:2181
    command:
      - bash
      - -c
      - >
        echo "Installing Connector"

        confluent-hub install --no-prompt debezium/debezium-connector-mysql:1.7.0

        confluent-hub install --no-prompt confluentinc/kafka-connect-elasticsearch:14.1.0

        confluent-hub install --no-prompt neo4j/kafka-connect-neo4j:2.0.0

        #

        echo "Launching Kafka Connect worker"

        /etc/confluent/docker/run &

        #

        sleep infinity
    volumes:
      - ./kafka-connect-plugins:/etc/kafka-connect/jars
    networks:
      - kafka-net
networks:
  kafka-net:
    driver: bridge

```

```bash
#Lưu ý cài package mới nhất
confluent-hub install --no-prompt confluentinc/kafka-connect-elasticsearch:14.1.0
```



Tạo connect
```bash
curl --location 'http://192.168.18.128:8083/connectors' \
--header 'Content-Type: application/json' \
--data '{
  "name": "elasticsearch-sink-connector",
  "config": {
    "connector.class": "io.confluent.connect.elasticsearch.ElasticsearchSinkConnector",
    "tasks.max": "1",
    "topics": "topic-log-1",
    "connection.url": "http://192.168.18.128:9200",
    "connection.username": "elastic",
    "connection.password": "Provanhung77",
    "type.name": "_doc",
    "key.ignore": "true",
    "schema.ignore": "true",
    "transforms.key.type": "org.apache.kafka.connect.transforms.ExtractField$Key",
    "transforms.key.field": "id",
    "key.converter": "org.apache.kafka.connect.storage.StringConverter",
    "value.converter": "org.apache.kafka.connect.json.JsonConverter",
    "value.converter.schemas.enable": "false"
  }
}'
```

Xem connect status

```bash

http://0.0.0.0:8083/connectors/elasticsearch-sink-connector/status

curl -X GET http://localhost:8083/connector-plugins

curl --location --request DELETE 'http://34.135.32.57:8083/connectors/elasticsearch-sink-connector'

curl -X GET http://localhost:8083/connectors/my-elasticsearch-connector/status

curl -X GET http://localhost:8083/connectors/my-elasticsearch-connector/config
```


Xem dữ liệu kafka
```bash
http://34.135.32.57:9200/spf-clean-log-all/_search
```
