## Cấu hình CICD sorce với .NET 8 k8s

Cấu hình file `.gitlab-ci.yml` như sau:
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
  SSH_HOST: 34.135.32.57
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
    - cat $SSH_KEY
    - ssh-keygen -y -f $SSH_KEY
    - |
      ssh -v -o StrictHostKeyChecking=no -p $SSH_PORT -i $SSH_KEY $SSH_USER@$SSH_HOST << EOF

      kubectl get pods

      kubectl set image deployment/spf-clean-deployment spf-clean=$DOCKER_IMAGE

      EOF
```

Tạo CICD Variables `Type là file` và biến tên là SSH_KEY   

Tạo key và cấu hình SSH key ở mục Ubuntu:

Kiểm tra key hợp lệ chưa ?

```bash
Key hợp lệ fortmat như sau:

-----BEGIN OPENSSH PRIVATE KEY-----
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
-----END OPENSSH PRIVATE KEY-----


# Cách kiểm tra
Vào notepad ++ > View > Sơ Symbol > Show End of Line


Tham khảo link sau Key phải có Enter cuối dòng

Tìm trên GG: SSH_KEY error in libcrypto

https://unix.stackexchange.com/questions/577402/ssh-error-while-logging-in-using-private-key-loaded-pubkey-invalid-format-and
```

## Tạo docker file:

```bash

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
ENTRYPOINT ["dotnet", "client-web-api.dll"]
```

## Cấu hình file deployment

```bash
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: spf-clean-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: spf-clean
  template:
    metadata:
      labels:
        app: spf-clean
    spec:
      containers:
      - name: spf-clean
        image: vanhungdev/spf-clean-hungnv165:435b7e85
        resources:
          requests:
            memory: "1.5Gi"
          limits:
            memory: "1.5Gi"
        ports:
        - containerPort: 5002

---
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: spf-clean-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: spf-clean-deployment
  minReplicas: 3
  maxReplicas: 5
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 50

---
apiVersion: v1
kind: Service
metadata:
  name: spf-clean-service
spec:
  type: NodePort
  selector:
    app: spf-clean
  ports:
    - port: 80
      targetPort: 5002

```