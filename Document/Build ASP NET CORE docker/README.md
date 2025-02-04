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

FROM mcr.microsoft.com/dotnet/aspnet:8.0 AS base
USER app
WORKDIR /app
EXPOSE 8080
EXPOSE 8081

FROM mcr.microsoft.com/dotnet/sdk:8.0 AS build
ARG BUILD_CONFIGURATION=Release
WORKDIR /src


COPY ["client-web-api/client-web-api.csproj", "client-web-api/"]
COPY ["Application/Application.csproj", "Application/"]
COPY ["Domain/Domain.csproj", "Domain/"]
COPY ["SharedKernel/SharedKernel.csproj", "SharedKernel/"]
COPY ["Infrastructure/Infrastructure.csproj", "Infrastructure/"]
COPY ["Persistence/Persistence.csproj", "Persistence/"]



RUN dotnet restore "Application/Application.csproj"
RUN dotnet restore "Domain/Domain.csproj"
RUN dotnet restore "SharedKernel/SharedKernel.csproj"
RUN dotnet restore "Persistence/Persistence.csproj"
RUN dotnet restore "Infrastructure/Infrastructure.csproj"
RUN dotnet restore "client-web-api/client-web-api.csproj"


COPY . .
WORKDIR "/src/client-web-api"
RUN dotnet build "client-web-api.csproj" -c $BUILD_CONFIGURATION -o /app/build

FROM build AS publish
ARG BUILD_CONFIGURATION=Release
RUN dotnet publish "client-web-api.csproj" -c $BUILD_CONFIGURATION -o /app/publish /p:UseAppHost=false

FROM base AS final
WORKDIR /app
COPY --from=publish /app/publish .

ARG ASPNETCORE_ENVIRONMENT=Development
ENV ASPNETCORE_ENVIRONMENT=${ASPNETCORE_ENVIRONMENT}

ENTRYPOINT ["dotnet", "client-web-api.dll"]

```

Lưu ý: nhớ config biến môi trường


```bash
ARG ASPNETCORE_ENVIRONMENT=Development
ENV ASPNETCORE_ENVIRONMENT=${ASPNETCORE_ENVIRONMENT}
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
vim appsettings.Development.json
 ```

Cách dùm vim để chỉnh sửa file:
  - Sa khi cấu hình xong file docker compose xong thì nhấn: `esc + :wq` để lưu và thoát.  
  - Phím `i` để chỉnh sửa.  
  - Phím `gg` để đưa con trỏ chuột về đầu file.   
  - Phím `ggdG` để xoá toàn bộ file.  
  - Phím `dG` để xoá từ vị trí con trỏ chuột về cuối file và lưu ý `G viết hoa`.  




## Phần 2: Chạy source .NET Core với VS code:  


```bash
# CD vào thư mục source api
cd source-api
```

Chạy với profile cụ thể

```bash
dotnet run --launch-profile Development
```

kill port

```bash

netstat -vanp tcp | grep 5000
killall -9 [tên tiến trình]

```
