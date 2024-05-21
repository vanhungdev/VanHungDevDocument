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

 ```bash

# lưu ý nó dùng password của container cũ
docker run -d --name sql-server-container -p 1433:1433 vanhungdev/sql-server-container_backup:v.16.01.2024

 ```

## Phần 2: Backup và chuyển tất cả các container cũ
### backup với bash script và cronjob


```bash


#!/bin/bash

# Thiết lập múi giờ UTC+7
export TZ="Asia/Ho_Chi_Minh"

# Lấy ngày giờ hiện tại (đã điều chỉnh về UTC+7)
TIMESTAMP=$(date +%Y.%m.%d-%H.%M.%S)

# In ra thời gian để kiểm tra
echo "Thời gian hiện tại (UTC+7): $TIMESTAMP"

# Danh sách các container cần backup
CONTAINERS=(
    "es01" # Elasticsearch
    "mongodb" # MongoDB
	"minio-server" # Minio
	"redis" # Redis
	"kib01"
)

# Tên image trên Docker Hub (thay đổi nếu cần)
DOCKER_IMAGE_PREFIX="vanhungdev"

# Thông tin đăng nhập Docker Hub (thay đổi thông tin của bạn)
DOCKER_USERNAME="vanhungdev"
DOCKER_PASSWORD="Provanhung77"

# Tên file log
LOG_FILE="backup_log.txt"

# Hàm backup một container
backup_container() {
    CONTAINER_NAME=$1
    IMAGE_NAME="$DOCKER_IMAGE_PREFIX/$CONTAINER_NAME"

    # Commit container
    docker commit "$CONTAINER_NAME" "$IMAGE_NAME:$TIMESTAMP"
    echo "$(date) - Backup $CONTAINER_NAME thành công: $IMAGE_NAME:$TIMESTAMP" >> "$LOG_FILE"

    # Tag image (nếu cần)
    docker tag "$IMAGE_NAME:$TIMESTAMP" "$IMAGE_NAME:latest"
    echo "$(date) - Tag $IMAGE_NAME:latest thành công" >> "$LOG_FILE"

    # Push image lên Docker Hub
    docker push "$IMAGE_NAME:$TIMESTAMP"
    echo "$(date) - Push $IMAGE_NAME:$TIMESTAMP thành công" >> "$LOG_FILE"
    docker push "$IMAGE_NAME:latest"
    echo "$(date) - Push $IMAGE_NAME:latest thành công" >> "$LOG_FILE"
}

# Đăng nhập vào Docker Hub
echo "$(date) - Đang đăng nhập vào Docker Hub..." >> "$LOG_FILE"
docker login -u "$DOCKER_USERNAME" -p "$DOCKER_PASSWORD"

# Backup từng container
for CONTAINER_NAME in "${CONTAINERS[@]}"; do
    echo "$(date) - Đang backup $CONTAINER_NAME..." >> "$LOG_FILE"
    backup_container "$CONTAINER_NAME"
done

echo "$(date) - Hoàn thành backup tất cả container!" >> "$LOG_FILE"

```


Cấp quyền.    

 ```bas
sudo chmod +x ./backup-container.sh

 ```

 Chạy file.    

 ```bas
 sudo ./backup-container.sh

 ```