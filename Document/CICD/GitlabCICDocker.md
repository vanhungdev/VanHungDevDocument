
## Cấu hình CICD với GITLAB Docker VPS Centos server

Để cấu hình được CI/CD ASP.NET Docker-compose chạy auto khi merge code chúng ta cần các bước sau:

1: Docker file
    - File docker trong project phải đúng có thể build được image không lỗi ở local

2: Tạo một server VPS
    - Chúng ta deploy bằng cách cài docker nên server phải có docker
    - Có thể SSH bằng SSH key

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
  SSH_HOST: 34.125.122.249
  SSH_PORT: 22
  SSH_PASSWORD: Provanhung77

before_script:
  - ls

build:
  stage: build
  image: docker:latest
  services:
    - docker:dind
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
      docker stop ParkingSystemClient || true
      docker rm ParkingSystemClient || true
      docker run -p 5005:5005 -d --name ParkingSystemClient $DOCKER_IMAGE
      EOF

```

Chú ý: 
Ở đây sẽ có 3 bước
  - build
  - test
  - deploy

`build` Là bước mà hệ thống CI/CD sẽ build `IMAGE` từ docker file. Bước này yêu cầu có docker và phải login vào docker hub cá nhân.

`test` Bước này test code. Tàm thời bypass

`deploy` Bước này đơn giản là hệ thống sẽ ssh vào server VPS centos của chúng ta để chạy các lênh để update lại image version mới nhất

*Chú ý:*   
    - Phải cấp quyền 400 cho file SSH Key khác thì hệ thống sẽ không nhận  
    - Viết các lệnh docker gắn liền với lệnh SSH theo cú pháp.
