# Triển khai ứng dựng ASP.NET 7 trên môi trường container VPS Centos 7x
## Phần 1: Deploy ứng dụng .NET lên VPS:  

**Để deploy được úng dụng .NET 7 lên chúng ta cần có các bước sau:**   

1. **Tạo docker file** - Để chạy các SDK cần thiết.
2. **Build image** - Chúng ta sẽ build image từ docker file.
3. **Run và test ở local** - test website trước ở localhost kiểm tra các file tài nguyên đã được chưa.
4. **Đẩy Repositories lên docker hub** - Công cụ theo dõi và quản lý cần thiết cho việc load test.
5. **Run và test trên VPS Centos** - chạy thử ở môi trường thật.
6. **Cấu hình Reverse Proxy, Load Balance, trỏ domain và Let's Encrypt SSL** - chạy thử ở môi trường thật.

**1. Tạo docker file**  
Tạo file docker file có tên `dockerfile` đặt ở đường dẫn có file .sln như sau:  


```bash
# Sử dụng hình ảnh cơ sở .NET Core Runtime
FROM mcr.microsoft.com/dotnet/aspnet:7.0 AS base
WORKDIR /app
EXPOSE 80
EXPOSE 443

# Sử dụng hình ảnh cơ sở .NET Core SDK để xây dựng ứng dụng
FROM mcr.microsoft.com/dotnet/sdk:7.0 AS build
WORKDIR /src

# Copy csproj và restore các phụ thuộc
# Lưu ý phải copy đầy đủ các project con các tầng Application, Infrastructure nếu có
COPY ["BlogHung/BlogHung.csproj", "BlogHung/"]
COPY ["BlogHung.Application/BlogHung.Application.csproj", "BlogHung.Application/"]
COPY ["BlogHung.Infrastructure/BlogHung.Infrastructure.csproj", "BlogHung.Infrastructure/"]
RUN dotnet restore "BlogHung/BlogHung.csproj"
RUN dotnet restore "BlogHung.Application/BlogHung.Application.csproj" 
RUN dotnet restore "BlogHung.Infrastructure/BlogHung.Infrastructure.csproj"

# Copy toàn bộ mã nguồn và build ứng dụng
COPY . .
WORKDIR "/src/BlogHung"
RUN dotnet build "BlogHung.csproj" -c Release -o /app/build

# Build runtime image
FROM build AS publish   
RUN dotnet publish "BlogHung.csproj" -c Release -o /app/publish

# Chọn hình ảnh cơ sở để chạy ứng dụng
FROM base AS final
WORKDIR /app
COPY --from=publish /app/publish .
ENTRYPOINT ["dotnet", "BlogHung.dll"]

```

**2. Build image**  
   
  **Lưu ý:** Để trình chạy docker file không gặp lỗi thì chú ý các mục sau:  

1. Chú ý `Cấu trúc thư mục` .  
2. Chú ý `COPY và  restore` đầy đủ các project.  
3. Chú ý `base Image SDK, Runtime` đã đúng version hay chưa.  

Mở terminal và di chuyển đến thư mục chứa tệp Dockerfile, sau đó chạy lệnh:  
 ```bash
  docker build --platform linux/amd64 -t imageName --no-cache .
 ```
Lưu ý: nếu chạy ở local thì không cần `linux/amd64` cái này chỉ để deploy lên VPS Centos

**3. Run và test ở local**  
  Run container:  
 
 ```bash
  docker run -p 44388:80 --name containerName  imageName
 ```
**4. Đẩy image kên docker hub**  
  Login vào docker hub:  
 
 ```bash
  dokcer login
 ```

  Đánh tag để đưa lên docker hub:  
 
 ```bash
  docker tag imageName:latest vanhungdev/imageName:v.1.1
 ```

  Đẩy image lên docker hub:  
 
 ```bash
  docker push vanhungdev/imageName:v.1.1
 ```

**5. Run và test trên VPS Centos**  
  Run container trên môi trường VPS Centos:  
 
 ```bash
  docker run -p 44380:80 --name containerName vanhungdev/imageName:v.1.1 
 ```
**6. Cấu hình Reverse Proxy, Load Balance, trỏ domain và Let's Encrypt SSL**  

Phần này xem ở phần nginx


**7. Cách chỉnh appsettings.Development.json trên môi trường container**  

bin vào môi trương container:  

 ```bash
  docker exec -it <container_id> /bin/sh 
 ```

Cài đặt vi hoặc nano để chỉnh sửa file
```bash
   apt-get update
   apt-get install -y vim
   apt-get install -y nano
 ```

Sửa file:

```bash
vim /src/appsettings.Development.json
 ```

Sửa file:
  - Sa khi cấu hình xong file docker compose xong thì nhấn: `esc + :wq` để lưu và thoát.  
  - Phím `i` để chỉnh sửa.  
  - Phím `gg` để đưa con trỏ chuột về đầu file.   
  - Phím `ggdG` để xoá toàn bộ file.  
  - Phím `dG` để xoá từ vị trí con trỏ chuột về cuối file và lưu ý `G viết hoa`.  

