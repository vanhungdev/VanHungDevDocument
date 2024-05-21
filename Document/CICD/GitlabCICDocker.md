
## Cấu hình CICD với GITLAB Docker VPS Centos server

Để cấu hình được CI/CD ASP.NET Docker-compose chạy auto khi merge code chúng ta cần các bước sau:

1: Docker file
    - File docker trong project phải đúng có thể build được image không lỗi ở local

2: Tạo một server VPS
    - Chúng ta deploy bằng cách cài docker nên server phải có docker
    - Có thể SSH bằng SSH key
    Truy cập vào file config của ssh trên server chỉnh cấu hình như sau
     ```bash
       vi /etc/ssh/sshd_config
     ```
    
    Diều chỉnh 2 dòng config
    
     ```bash
      PubkeyAuthentication yes
      AuthorizedKeysFile .ssh/authorized_keys
    
     ```


3: Thêm biến môi trường ở Settings > CI/CD
    -  sau khi config server có thể SSH băng Key thì cần tạo biến lưu giá trị của public key

4: tạo file .gitlab-ci.yml cùng cấp với file .snl

```bash
image: mcr.microsoft.com/dotnet/sdk:8.0

stages:
  - build
  - test
  - deploy

variables:
  DOCKER_IMAGE: vanhungdev/$CI_PROJECT_NAME:$CI_COMMIT_SHORT_SHA
  DOCKER_USERNAME: vanhungdev
  DOCKER_PASSWORD: Provanhung77
  SSH_USER: root
  SSH_HOST: 35.222.111.241
  SSH_PORT: 22
  SSH_PASSWORD: Provanhung77

before_script:
  - ls

build:
  stage: build
  image: docker:20.10.16
  services:
    - docker:20.10.16-dind
  script:
    - docker login -u $DOCKER_USERNAME -p $DOCKER_PASSWORD
    - docker build -t $DOCKER_IMAGE --build-arg BUILD_CONFIGURATION=Release .
    - docker push $DOCKER_IMAGE

test:
  stage: test
  script:
    - ls

deploy:
  stage: deploy
  script:
    - apt-get update && apt-get install -y openssh-client
    - mkdir -p ~/.ssh
    - chmod 400 $SSH_KEY
    - |
      ssh -o StrictHostKeyChecking=no -p $SSH_PORT -i $SSH_KEY $SSH_USER@$SSH_HOST << EOF
      docker ps
      docker pull $DOCKER_IMAGE
      docker stop ParkingSystemScanner || true
      docker rm ParkingSystemScanner || true
      docker run -p 5004:5004 -d --name ParkingSystemScanner $DOCKER_IMAGE

      docker stop systemscannerssl || true
      docker rm systemscannerssl || true

      docker run -d --name systemscannerssl \
          --restart=always \
          -e VIRTUAL_HOST="hi-static-spf.site" \
          -e VIRTUAL_PORT=5004 \
          -e LETSENCRYPT_HOST="hi-static-spf.site" \
          -e LETSENCRYPT_EMAIL="vanhungdev@fpt.com.vn" \
          $DOCKER_IMAGE
      EOF


```

Chú ý: 
Ở đây sẽ có 3 bước
  - build
  - test
  - deploy

Có thể chỉ định các stage chạy branch nào  
Ví dụ stage `test` chỉ chạy với branch developer

```bahs

test:
  stage: test
  only:  # Chỉ chạy job này trên các branch phát triển
    - develop
    - /^feature-.*/  # Regex để match các branch bắt đầu với 'feature-'
  script:
    - ls

```

`build` Là bước mà hệ thống CI/CD sẽ build `IMAGE` từ docker file. Bước này yêu cầu có docker và phải login vào docker hub cá nhân.

`test` Bước này test code. Tàm thời bypass

`deploy` Bước này đơn giản là hệ thống sẽ ssh vào server VPS centos của chúng ta để chạy các lênh để update lại image version mới nhất

*Chú ý:*   
    - Phải cấp quyền 400 cho file SSH Key khác thì hệ thống sẽ không nhận  
    - Viết các lệnh docker gắn liền với lệnh SSH theo cú pháp.

## Gitlab Runner

1. Cài đặt runner
```bash
docker run -d --name gitlab-runner --restart always     -v /srv/gitlab-runner/config:/etc/gitlab-runner     -v /var/run/docker.sock:/var/run/docker.sock     gitlab/gitlab-runner:v15.8.2

```

2. Khởi tạo một runner mới
```bash

docker exec -it gitlab-runner gitlab-runner register

```


3. Nhập các thông tin cần thiết

```bash

Các thông tin cần nhập vào như sau:

        3.1 Nhập URL https://gitlab.com/
        3.2 Nhập token ở runner trong project set (setting > CICD > Runner).
        3.3 Nhập mô tả runner.
        3.4 Gắn tags cho runner có thể để trống
        3.5 Nhập docker.
        3.6 Nhập một image mặc định cho Docker image

```


4. docker restart gitlab-runner
```bash

docker restart gitlab-runner

```

5. Hiển thị các runner
```bash

docker exec -it gitlab-runner gitlab-runner list

```

6. Stopping, Starting và Restarting Runners
```bash

docker exec -it gitlab-runner gitlab-runner stop

docker exec -it gitlab-runner gitlab-runner start

docker exec -it gitlab-runner gitlab-runner restart
```

7. Huỷ, Xoá Runner
```bash

docker exec -it gitlab-runner gitlab-runner unregister --name a9a3dd7b82bd

docker exec -it gitlab-runner gitlab-runner verify --delete

```


## Phân Loại Gitlab Runner

### Các loại gitlab runner

Có hai loại Gitlab Runner chính được phân loại dựa trên cách thức quản lý:

#### 1. Shared Runners:

Cung cấp bởi GitLab:  
Miễn phí cho dự án Public và giới hạn 400 phút/tháng cho Private. (Nếu cần dùng thêm cần mua thêm)
Dễ sử dụng, không cần cài đặt hay cấu hình.
Phù hợp cho các dự án nhỏ, cá nhân hoặc mới bắt đầu.  

Hạn chế:  
Ít tùy chỉnh, ít linh hoạt, không phù hợp cho nhu cầu cao.
Có thể quá tải vào giờ cao điểm.  

#### 2. Self-Managed Runners:

Do người dùng cài đặt và quản lý:  
Cài đặt trên server riêng hoặc cloud instance.  Tùy chỉnh cao, linh hoạt, đáp ứng mọi nhu cầu.  Miễn phí hoàn toàn. 

Yêu cầu:  
Kiến thức quản trị hệ thống, cài đặt, cấu hình.
Chịu trách nhiệm bảo trì, cập nhật.
Phù hợp cho dự án lớn, có đội ngũ DevOps.
Ngoài ra, Gitlab Runner còn được phân loại theo executor 


Cách thức thực thi các job:  

`Shell:` Chạy lệnh shell trên môi trường runner.  
`SSH:` Chạy lệnh shell trên máy chủ khác qua SSH.  
`Docker:` Chạy container Docker.  
`Kubernetes:` Chạy job trên Kubernetes cluster.  
`VirtualBox:` Chạy job trong máy ảo VirtualBox.  
`Custom:` Tự định nghĩa cách thức thực thi job.  
Lựa chọn loại Gitlab Runner nào phụ thuộc vào nhu cầu,nguồn lực và kiến thức kỹ thuật của bạn.  

#### Tóm lại:

`Shared Runners:` Miễn phí, dễ sử dụng, phù hợp cho nhu cầu cơ bản.
Self-Managed Runners: Linh hoạt, tùy chỉnh cao, phù hợp cho nhu cầu cao, phức tạp.
