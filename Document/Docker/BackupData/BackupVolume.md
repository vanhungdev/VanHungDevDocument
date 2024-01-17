# Backup Volume  

Trong phần này sẽ hướng dẫn tất tần tật về Backup Volume trên docker


## Phần 1: Cài đặt docker trên Centos 7  

**Build image từ container và đẩy lên docker hub sau đó build image**  

 ```bash

Step 1. Xác định volumes đang được sử dụng bởi container Elasticsearch.  
Step 2. Đánh tag cho images.  
Step 3. Sao chép volumes đó sang server đích (thay IP và đường dẫn phù hợp).  
Step 4. Trên server đích, tạo volumes mới với tên tương ứng, ví dụ.
Step 5. Chạy container Elasticsearch mới với volumes vừa copy.
```


Step 1. Xác định volumes đang được sử dụng bởi container Elasticsearch.  

 ```bash
# 
docker inspect es01 | grep -i volume
 ```

Step 3. Đánh tag cho images.  

 ```bash
# 
docker stop es01
 ```


Step 3. Sao chép volumes đó sang server đích (thay IP và đường dẫn phù hợp)

 ```bash
# 
scp -r /var/lib/docker/volumes/es01-volume/_data root@34.170.130.200:/var/lib/docker/volumes/
 ```


Step 4. Trên server đích, tạo volumes mới với tên tương ứng, ví dụ.  

 ```bash
# 
docker volume create --name es01-volume
 ```


Step 5. Chạy container Elasticsearch mới với volumes vừa copy.  

 ```bash
# 
docker run --name es02 --net elastic -p 9200:9200 -p 9300:9300 -v es01-volume:/usr/share/elasticsearch/data elasticsearch:7.10.2
 ```

