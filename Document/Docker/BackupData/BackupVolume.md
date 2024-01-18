# Backup Volume  

Trong phần này sẽ hướng dẫn tất tần tật về Backup Volume trên docker


## Phần 1: Cài đặt docker trên Centos 7  

**Build image từ container và đẩy lên docker hub sau đó build image**  

 ```bash
# Xóa dữ liệu trên máy đích
rm -rf /var/lib/docker/volumes/es-data/_data
```
 ```bash
# Tạo các thư mục ở server đích trước khi đẩy dữ liệu server đích
mkdir -p /var/lib/docker/volumes/mongodb-data/
mkdir -p /var/lib/docker/volumes/mssql-data/
mkdir -p /var/lib/docker/volumes/es-data/
```
 ```bash
#Chuyển dữ liệu từ server nguồn
scp -r /var/lib/docker/volumes/mongodb-data/ root@34.132.18.83:/var/lib/docker/volumes/mongodb-data/
scp -r /var/lib/docker/volumes/mssql-data/ root@34.132.18.83:/var/lib/docker/volumes/mssql-data/
scp -r /var/lib/docker/volumes/es-data/ root@34.132.18.83:/var/lib/docker/volumes/es-data/
```
 ```bash
# Chạy các container với volume trên server đích 
docker run -d --name mongodb -p 27017:27017 -v mongodb-data:/data/db -e MONGO_INITDB_ROOT_USERNAME=hungnv165 -e MONGO_INITDB_ROOT_PASSWORD=Provanhung77 mongo:latest
docker run -d --name sql-server -p 1433:1433 -v mssql-data:/var/opt/mssql -e ACCEPT_EULA=Y -e SA_PASSWORD=Provanhung77 mcr.microsoft.com/mssql/server:2017-latest
docker run --name es01 --net elastic -p 9200:9200 -p 9300:9300 -v es-data/_data:/usr/share/elasticsearch/data -e "discovery.type=single-node" -e "xpack.security.enabled=true" -e "ELASTIC_PASSWORD=Provanhung77" -e "xpack.security.http.ssl.enabled=false" elasticsearch:8.11.3
```


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


