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
  docker run --name es01 --net elastic -p 9200:9200 -p 9300:9300 -e "discovery.type=single-node" -e "network.host=0.0.0.0"  -e "xpack.security.enabled=true" -e "ELASTIC_PASSWORD=Provanhung77" -e "xpack.security.http.ssl.enabled=false" -t docker.elastic.co/elasticsearch/elasticsearch:8.11.3 --restart=always 

 ```

note: sau khi cài xong có thể dùng Provanhung77 để đăng nhập, chúng ta đã cấu hình ở trên

**3. Lấy password của tài khoản kibana_system**  
 
 ```bash
docker exec -it es01 /usr/share/elasticsearch/bin/elasticsearch-reset-password -u kibana_system

 ```
 Nếu vào exec vào rồi hoặc dùng portainer thì chỉ cần 

  ```bash
/usr/share/elasticsearch/bin/elasticsearch-reset-password -u kibana_system

 ```

**4. Chạy kibna container**  
 
 ```bash
  docker run --name kib01 --net elastic -p 5601:5601 docker.elastic.co/kibana/kibana:8.11.3 --restart=always 
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

Đẩy dữ liệu và thêm index

 ```bash
curl -u "elastic:Provanhung77" -X POST "http://your_ip_address:9200/name_index1/_doc" -H 'Content-Type: application/json' -d '{
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
  }'
 ```


**Dữ liệu mẫu để thao tác search**  

```bash
  POST products/_bulk
  {"index":{"_id":4}}
  {"name":"Product 4","desc":"Description 4","price":129,"stock":42,"sale":false,"created_at":"2023-01-04T17:22:08Z"}
  {"index":{"_id":14}}
  {"name":"Product 14","desc":"Description 14","price":290,"stock":4,"sale":true,"created_at":"2023-02-22T00:09:58Z"} 
  {"index":{"_id":15}}
  {"name":"Product 15","desc":"Description 15","price":420,"stock":10,"sale":false,"created_at":"2023-02-26T06:33:29Z"}
  {"index":{"_id":16}}
  {"name":"Product 16","desc":"Description 16","price":340,"stock":12,"sale":false,"created_at":"2023-02-27T04:56:11Z"}
  {"index":{"_id":17}}
  {"name":"Product 17","desc":"Description 17","price":260,"stock":8,"sale":true,"created_at":"2023-02-28T22:13:32Z"}
  {"index":{"_id":18}}
  {"name":"Product 18","desc":"Description 18","price":180,"stock":23,"sale":false,"created_at":"2023-01-01T15:47:24Z"}
  {"index":{"_id":19}}
  {"name":"Product 19","desc":"Description 19 good","price":120,"stock":17,"sale":true,"created_at":"2023-01-05T03:18:45Z"}
  {"index":{"_id":20}}
  {"name":"Product 20","desc":"Description 20 bad","price":220,"stock":11,"sale":false,"created_at":"2023-01-10T12:41:16Z"}
  {"index":{"_id":21}}
  {"name":"Product new york city","desc":"Description new york city","price":220,"stock":11,"sale":false,"created_at":"2023-01-10T12:41:16Z"}

 ```
