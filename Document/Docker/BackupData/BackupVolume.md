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






 ```bash

docker run -d --name mongodb -p 27017:27017 -v mongodb-data:/data/db -e MONGO_INITDB_ROOT_USERNAME=hungnv165 -e MONGO_INITDB_ROOT_PASSWORD=Provanhung77 mongo:latest

docker run -d --name sql-server -p 1433:1433 -v mssql-data:/var/opt/mssql -e ACCEPT_EULA=Y -e SA_PASSWORD=Provanhung77 mcr.microsoft.com/mssql/server:2017-latest

docker run --name es01 --net elastic -p 9200:9200 -p 9300:9300 -v es-data:/usr/share/elasticsearch/data -e "discovery.type=single-node" -e "xpack.security.enabled=true" -e "ELASTIC_PASSWORD=Provanhung77" -e "xpack.security.http.ssl.enabled=false" elasticsearch:8.11.3

hướng dẫn dùng scp để chuyển dữ liệu các volume trên

 ```




 ```bash
# 
Để sử dụng scp chuyển dữ liệu từ các docker volume của MongoDB, SQL Server và Elasticsearch, bạn có thể làm như sau:

Xác định đường dẫn tới các volume trên host:
MongoDB: /var/lib/docker/volumes/mongodb-data/_data
SQL Server: /var/lib/docker/volumes/mssql-data/_data
Elasticsearch: /var/lib/docker/volumes/es-data/_data
Trên host chứa các volume, chạy scp để copy dữ liệu tới host khác:

 ```

scp -r /var/lib/docker/volumes/mongodb-data/_data user@host:/backup

scp -r /var/lib/docker/volumes/mssql-data/_data user@host:/backup  

scp -r /var/lib/docker/volumes/es-data/_data user@host:/backup



Trên host đích, kiểm tra dữ liệu đã được copy thành công tới thư mục /backup.  
Để tự động hóa, bạn có thể tạo script với các lệnh scp trên, sau đó dùng crontab chạy script theo định kỳ.  
Như vậy là đã sử dụng được scp để sao lưu và chuyển dữ liệu từ các docker volume sang một host khác.  

1 Tạo file sao_luu.sh với nội dung:

 ```bash
#!/bin/bash

# Thông tin host đích
DEST_HOST=backup_server
DEST_DIR=/backup

# Sao lưu volume mongodb
mongo_data=/var/lib/docker/volumes/mongodb-data/_data
scp -r $mongo_data user@$DEST_HOST:$DEST_DIR

# Sao lưu volume sqlserver 
sql_data=/var/lib/docker/volumes/mssql-data/_data
scp -r $sql_data user@$DEST_HOST:$DEST_DIR

# Sao lưu volume elasticsearch
es_data=/var/lib/docker/volumes/es-data/_data
scp -r $es_data user@$DEST_HOST:$DEST_DIR

```

2. Phân quyền thực thi cho file sao_luu.sh:
chmod +x sao_luu.sh

3. Chạy thử script với lệnh:
./sao_luu.sh

4. Kiểm tra dữ liệu đã copy sang máy đích thành công.

5. Thiết lập crontab để chạy script định kỳ, ví dụ mỗi ngày vào lúc 0h:
   Mở crontab editor:
    crontab -e

   thêm caua lệnh
0 0 * * * /path/to/sao_luu.sh

Như vậy script sẽ tự động sao lưu dữ liệu từ các volume ra máy chủ đích hàng ngày. Bạn có thể chỉnh sửa script để sao lưu linh hoạt hơn.


