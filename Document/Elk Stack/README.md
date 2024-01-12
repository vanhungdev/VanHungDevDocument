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

## Phần 3: Một số câu query elastic cơ bản: 


**1. Match và multi_match Query**

`match:` Tìm kiếm full text. Cho phép tìm kiếm văn bản tự do, tìm các document có chứa từ khóa. Sử dụng analyzer để tokenize văn bản.  
`multi_match:` Cho phép match trên nhiều trường cùng lúc thay vì 1 trường. Có thể kết hợp với các loại query khác như match, match_phrase, etc.  

**Tìm tất cả field theo value (multi_match)**  
Search Lite API  

```bash
  GET /index1/_search?q=a0aff37f-5720-4d49-b2ea-0d91ff26f1ac
 ```


Full JSON request body
Nếu chúng ta không để `fields` thì sẽ tìm tất cả file, còn nếu để thì chỉ tìm trong phạm vị `fields`


```bash
  POST /index1/_search
  {
      "query": {
          "multi_match" : {
              "query" : "7f5c6f5f-bd57-4a71-bd7a-3037d4efcb44",
              "fields" : ["fields.CorrelationId.keyword", "fields.IpAddress.keyword"]
          }
      }
  }
 ```



Lọc kết quả trả về chỉ lấy `fields.LogFolder`, `fields.SourceContext`, `fields.ContentType`, lọc theo `size`, `from`
```bash
  POST /index1/_search
  {
    "query": {
      "multi_match": {
        "query": "7f5c6f5f-bd57-4a71-bd7a-3037d4efcb44",
        "fields": [
          "fields.CorrelationId.keyword",
          "fields.IpAddress.keyword"
        ]
      }
    },
    "size": 2,
    "from": 0,
    "_source": [
      "fields.LogFolder",
      "fields.SourceContext",
      "fields.ContentType"
    ],
    "highlight": {
      "fields": {
        "fields.LogFolder": {},
        "fields.SourceContext": {},
        "fields.ContentType": {}
      }
    }
  }


// Nếu kết hợp lại thì chỉ tìm trong `fields` và chỉ trả về trong `_source`
 ```

Gọi bằng cURL post man
```bash

  curl --location 'http://elastic:Jfs-zBo+NO4dhxVuJEpR@34.16.204.104:9200/index1/_search' \
  --header 'Content-Type: application/json' \
  --data '{
    "query": {
      "multi_match": {
        "query": "7f5c6f5f-bd57-4a71-bd7a-3037d4efcb44",
        "fields": [
          "fields.CorrelationId.keyword",
          "fields.IpAddress.keyword"
        ]
      }
    },
    "size": 2,
    "from": 0,
    "_source": [
      "fields.LogFolder",
      "fields.SourceContext",
      "fields.ContentType"
    ],
    "highlight": {
      "fields": {
        "fields.LogFolder": {},
        "fields.SourceContext": {},
        "fields.ContentType": {}
      }
    }
  }'
 ```


**Tìm theo field được chỉ định (match)**  
Search Lite API  

```bash
GET /index1/_search?q=fields.CorrelationId.keyword:30d2418f-0c51-44c8-98e3-a5cc06532c76
 ```


Full JSON request body
```bash
POST /index1/_search
 {
    "query": {
        "match": {
            "fields.CorrelationId.keyword": "30d2418f-0c51-44c8-98e3-a5cc06532c76"
        }
    },
    "size": 2,
    "from": 0,
    "_source": [
        "fields.LogFolder",
        "fields.SourceContext",
        "fields.ContentType"
    ],
    "highlight": {
        "fields": {
            "title": {}
        }
    }
}

 ```

Gọi bằng cURL post man
```bash
   curl --location 'http://elastic:Jfs-zBo+NO4dhxVuJEpR@34.16.204.104:9200/index1/_search' \
   --header 'Content-Type: application/json' \
   --data '{
       "query": {
           "match": {
               "fields.CorrelationId.keyword": "30d2418f-0c51-44c8-98e3-a5cc06532c76"
           }
       },
       "size": 2,
       "from": 0,
       "_source": [
           "fields.LogFolder",
           "fields.SourceContext",
           "fields.ContentType"
       ],
       "highlight": {
           "fields": {
               "title": {}
           }
       }
   }'

 ```

**2. Compound queries**  
1.Boolean query

Chuẩn bị dữ liệu test  

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

 ```

Lưu ý: thêm size để hiển thị thêm document

```bash
{
  "_source": true,
  "size": 100
}

 ```

```bash
  POST products/_search
  {
    "query": {
      "bool": {
        "must": [
          { "match": { "name": "product" }} 
        ],
        
        "filter": [
          { "range": { "price": { "gte": 100 }} }
        ],
        
        "should": [
          { "match": { "desc": "good" }}
        ],
        
        "must_not": [
          { "match": { "desc": "bad" }} 
        ]
      }
    }
  }

 ```

Boolean query trên có ý nghĩa:

`MUST:` Name phải chứa từ "product"  
`FILTER:` Giá phải >= 100  
`SHOULD:` Nên chứa từ "good" trong desc (không bắt buộc)  
`MUST_NOT:` Không chứa từ "bad" trong desc  
Kết quả trả về sẽ được đánh giá dựa trên:  

Bắt buộc phải thỏa mãn các điều kiện trong must và filter  
Nếu thỏa must và filter, sẽ ưu tiên document có chứa từ "good" trong should  
Loại bỏ các document chứa từ "bad" trong must_not  

2. Boosting query  

```bash
  POST products/_search
  {
    "_source": true,
    "size": 100,
    "query": {
      "boosting": {
        "positive": {
          "match": {
            "name": "Product" 
          }
        },
        "negative": {
          "match": {
            "desc": "good"
          }
        },
        "negative_boost": 0.2
      }
    }
  }

 ```

Boosting query bao gồm:  

`positive:` Điều kiện để tăng điểm cho document. Ở đây sử dụng bool must để kết hợp 2 điều kiện:   
`negative_boost:` Hệ số giảm điểm cho phần negative. Ở đây là 0.2, tức giảm 20% điểm.  

3. Constant score 

```bash
{
  "query": {
    "constant_score": {
      "filter": {
        "term": {
          "sale": true
        }
      },
      "boost": 5
    }
  }
}


 ```
Truy vấn constant score không tính đến độ liên quan của tài liệu. Nó chỉ gán điểm số cố định cho các tài liệu phù hợp với điều kiện lọc.  
Bạn có thể sử dụng truy vấn constant score để ưu tiên các tài liệu cụ thể, bất kể mức độ phù hợp của chúng.  
Bạn cũng có thể sử dụng truy vấn constant score để tạo các truy vấn có điểm số cố định, chẳng hạn như truy vấn để tìm tất cả các tài liệu được tạo gần đây.  


4. Constant score 

```bash



 ```

5. Function score query 

```bash
aa

 ```
aaa


