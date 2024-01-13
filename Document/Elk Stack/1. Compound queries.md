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

Chưa lĩnh hội được

 ```
Viết mô tả


5. Function score query 

```bash
Chưa lĩnh hội được

 ```
Viết mô tả


