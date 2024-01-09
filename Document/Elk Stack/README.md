# Cấu hình Elasticsearch và kibana
- Việc tạo một elasticsearch và kibana giúp chúng ta theo dõi hoạt động của hệ thống và fix bug một cách dễ dàng, tiện lợi

## Phần 1: Cài đặt ELK Stack:  

**Để cài được Elasticsearch, kibna chúng ta cần cài theo các bước sau:**   

1. **Tạo elastic network** - Cần thiết để kết nối elasticsearch với kibna.
2. **Tạo elastic-server container** - Chúng ta sẽ chạy container elasticsearch trước.
3. **Lấy password của tài khoản kibana_system** - Cần lấy password của tk kibana_system vì kibana sẽ không kết nối với tk elastic.
4. **Chạy kibna container** - Chúng ta cần chạy kiban container.
5. **Lấy code từ log kibana và kết nối với elasticsearch** - Chạy thử ở môi trường thật.

**1. Tạo elastic network:**  

 ```bash
  docker network create elastic
 ```
**Bỏ limit size của elasticsearch server:**  
 
 ```bash
sudo vim /etc/sysctl.conf
 ```
Chèn dòng cấu hình `vm.max_map_count=262144` bằng cách nhập i để chuyển vào chế độ chèn, sau đó nhập dòng cấu hình.  

Sau khi cài xong chạy lệnh
 
 ```bash
sudo sysctl -w vm.max_map_count=262144
 ```

**2. Tạo elastic-server container**  

  Tạo elastic-server container:  
 
 ```bash
  docker run --name elastic-server --net elastic -p 9200:9200 -p 9300:9300 -e "discovery.type=single-node" -e "network.host=0.0.0.0"  -e "xpack.security.enabled=true" -e "ELASTIC_PASSWORD=Provanhung77" -e "xpack.security.http.ssl.enabled=false" -t docker.elastic.co/elasticsearch/elasticsearch:8.11.3

 ```

**3. Lấy password của tài khoản kibana_system**  
 
 ```bash
docker exec -it elastic-server /usr/share/elasticsearch/bin/elasticsearch-reset-password -u kibana_system

 ```
 Nếu vào exec vào rồi hoặc dùng portainer thì chỉ cần 

  ```bash
/usr/share/elasticsearch/bin/elasticsearch-reset-password -u kibana_system

 ```

**4. Chạy kibna container**  
 
 ```bash
  docker run --name kib01 --net elastic -p 5601:5601 docker.elastic.co/kibana/kibana:8.11.3
 ```

**5. Lấy code từ log kibana và kết nối với elasticsearch**  
Lưu ý: cần xem log của kibana và lấy code  
elastic_server là http://elastic-server:9200 hoặc là ip của container name của elasticsearch là được

## Phần 2: Tạo và đánh index elastic: 
Tạo index:
Đoạn code này cần chạy trong mục dev tools

 ```bash
   http://your_ip_address:5601/app/dev_tools#/console
 ```
 
 ```bash
  PUT /name_index1
{
  "mappings": {
    "properties": {
      "@t": {"type": "date"},
      "@mt": {"type": "text"},
      "@r": {"type": "float"},
      "CorrelationId": {"type": "keyword"},
      "IpAddress": {"type": "ip"},
      "viewResultModel": {"type": "text"},
      "CustomLog": {"type": "text"},
      "Host": {"type": "keyword"},
      "Source": {"type": "keyword"},
      "timestamp": {"type": "date"},
      "DateTime": {"type": "text"},
      "SourceType": {"type": "keyword"},
      "Protocol": {"type": "keyword"},
      "Scheme": {"type": "keyword"},
      "ContentType": {"type": "keyword"},
      "Header": {"type": "object"},
      "LogFolder": {"type": "keyword"},
      "EndpointName": {"type": "keyword"},
      "RequestMethod": {"type": "keyword"},
      "RequestPath": {"type": "keyword"},
      "StatusCode": {"type": "integer"},
      "Elapsed": {"type": "float"},
      "SourceContext": {"type": "keyword"},
      "RequestId": {"type": "keyword"}
    }
  }
}
 ```

Đẩy dữ liệu vào index:
 
 ```bash
  POST /name_index1/_doc
{
  "@t": "2023-11-14T06:31:05.4412033Z",
  "@mt": "HTTP {RequestMethod} {RequestPath} responded {StatusCode} in {Elapsed:0.0000} ms",
  "@r": [427.1187],
  "CorrelationId": "276554f2-044c-4c3d-a622-2ee080f1ec05",
  "IpAddress": "::1",
  "viewResultModel": "null",
  "CustomLog": "",
  "Host": "localhost:5001",
  "Source": "1",
  "timestamp": "2023-11-14T13:31:05.4313672+07:00",
  "DateTime": "14-11-2023 13:31:05",
  "SourceType": "USER-LOG",
  "Protocol": "HTTP/1.1",
  "Scheme": "http",
  "ContentType": "text/html; charset=utf-8",
  "Header": [],
  "LogFolder": "bloghung.controllers.homecontroller.index",
  "EndpointName": "/",
  "RequestMethod": "GET",
  "RequestPath": "/",
  "StatusCode": 200,
  "Elapsed": 427.1187,
  "SourceContext": "Serilog.AspNetCore.RequestLoggingMiddleware",
  "RequestId": "0HMV4RD3E53ED:00000001"
}

 ```

**Cách 2 là call bằng postman**  

Thêm index

 ```bash
curl -u "elastic:Provanhung77" -X POST "http://localhost:9200/ten_index/_doc" -H 'Content-Type: application/json' -d '{
  "@t": "2023-11-14T06:31:05.4412033Z",
  "@mt": "HTTP {RequestMethod} {RequestPath} responded {StatusCode} in {Elapsed:0.0000} ms",
  "@r": [427.1187],
  "CorrelationId": "276554f2-044c-4c3d-a622-2ee080f1ec05",
  "IpAddress": "::1",

  // Other fields here

}'
 ```

Đẩy dữ liệu

 ```bash
curl -u "elastic:Provanhung77" -X POST "http://localhost:9200/ten_index/_doc" -H 'Content-Type: application/json' -d '{
  "@t": "2023-11-14T06:31:05.4412033Z",
  "@mt": "HTTP {RequestMethod} {RequestPath} responded {StatusCode} in {Elapsed:0.0000} ms",
  "@r": [427.1187],
  "CorrelationId": "276554f2-044c-4c3d-a622-2ee080f1ec05",
  "IpAddress": "::1",

  // Other fields here

}'
 ```

## Phần 3: Một số câu query elastic cơ bản: 
Viết cái gì đó ở đây
