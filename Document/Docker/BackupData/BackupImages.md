# Backup image bằng cách build images 

Trong phần này sẽ hướng dẫn tất tần tật về build image từ container và đẩy lên docker hub

## Phần 1: Build image từ container và đẩy lên docker hub  

**Build image từ container và đẩy lên docker hub**  

 ```bash
	Step 1. Commit container này thành một image mới.  
	Step 2. Đánh tag cho images.
	Step 3. Push lên docker hub.
	Step 4. Run trên server mới như bình thường.
```


Step 1. Commit container này thành một image mới.  

 ```bash
# Xem danh sách container
docker ps

# Xem danh sách images
docker images

# Commit container thành image
docker commit <container hiện tại> <tên image muốn build>:v.16.01.2024

# Câu lênh mẫu sẽ như sau
docker commit sql-server-container sql-server-container_backup

# login vào docker hub
docker login
 ```

Step 2. Đánh tag cho images.  

 ```bas

# Đánh tag cho images
docker tag sql-server-container_backup vanhungdev/sql-server-container_backup:v.16.01.2024

 ```

Step 3. Push lên docker hub.    

 ```bas
docker push vanhungdev/sql-server-container_backup:v.16.01.2024

 ```


Step 4. Run trên server mới như bình thường.  

 ```bas

# lưu ý nó dùng password của container cũ
docker run -d --name sql-server-container -p 1433:1433 vanhungdev/sql-server-container_backup:v.16.01.2024

 ```

## Phần 2: Backup và chuyển tất cả các container cũ

