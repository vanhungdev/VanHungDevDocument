# backup container dữ liệu container về máy local  

Trong phần này sẽ hướng dẫn backup container dữ liệu container về máy local


## Phần 1: Backup dữ liệu container về máy local

Cách backup container dữ liệu container về máy local.  

 ```bas

	Step 1. Tạo image backup từ container  
	Step 2. Save image ra file tarball  
	Step 3. Copy file tarball về máy local, ví dụ sử dụng scp  
	Step 4. Load image từ file tarball  
	Step 5. Kiểm tra image đã được tải  
	Step 6. Khởi chạy container từ image backup  

 ```
**Step 1. Tạo image backup từ container**

 ```bas
docker commit sql-server-container sql-server-container_backup:v.16.01.2024

 ```

**Step 2. Save image ra file tarball**

 ```bas
docker save sql-server-container_backup:v.16.01.2024 -o sql-server-backup_16_01_2024.tar

 ```

**Step 3. Copy file tarball về máy local, ví dụ sử dụng scp**

 ```bas
scp -r root@remote_server:/path/to/sql-server-backup_16_01_2024.tar ~/backups/

 ```

**Step 4. Load image từ file tarball**

 ```bas
docker load -i ~/backups/sql-server-backup_16_01_2024.tar

 ```

**Step 5. Kiểm tra image đã được tải**

 ```bas
docker images

 ```

**Step 6. Khởi chạy container từ image backup**

 ```bas
docker run -d --name sql-server -p 1433:1433 sql-server-container_backup:v.16.01.2024


 ```


