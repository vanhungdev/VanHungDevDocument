# Build image chuyển dữ liệu giữa các server (docker save) 

Trong phần này sẽ hướng dẫn tất tần tật về build image chuyển dữ liệu giữa các server


## Phần 1: Build image chuyển dữ liệu giữa các server  

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

## Phần 2: Backup và chuyển tất cả các container cũ

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

Đối với kibana không backup kiểu này không được thì cài lại container  


 ```bas
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





