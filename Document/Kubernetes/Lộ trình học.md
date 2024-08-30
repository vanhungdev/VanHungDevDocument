# Lộ trình học Kubernetes và Công nghệ liên quan (Cập nhật)

## Phần 1: Nền tảng Kubernetes (3-4 tuần)
1. Giới thiệu về Kubernetes
2. Kiến trúc Kubernetes
   - Kube-apiserver
   - Kube-controller-manager
   - Kubelet
   - Kube-proxy
   - Container runtime
3. Các khái niệm cơ bản:
   - Pods
   - ReplicationControllers (Phiên bản cũ của `replicaset`được giữ lại để chạy các ứng dụng cũ)
   - ReplicaSets (có bọ chọn selector linh hoạt hơn)
   - DaemonSets (Đảm bảo 1 pods sẽ được triển khai nên một node, không sử dụng chung với replicaset)
4. Workload Resources:
   - Deployments
   - StatefulSets
   - Jobs và CronJobs (`Job` là một công việc ví dụ như tạo một `pods` có thể cấu hình xóa pods khi xong job, CronJob thực chất là một đối tượng trong Kubernetes, có nhiệm vụ tạo ra các Job theo lịch trình định sẵn)
5. Services và Networking
6. Volumes và Storage:
   - PersistentVolumes
   - PersistentVolumeClaims
7. Configuration:
   - ConfigMaps
   - Secrets
8. Quản lý tài nguyên:
   - Resource Requests và Limits
   - Namespace và Resource Quotas
9. Auto Scaling:
   - Horizontal Pod Autoscaler
   - Vertical Pod Autoscaler

## Phần 2: Triển khai và Quản lý Kubernetes (3-4 tuần)
1. Thiết lập cụm Kubernetes:
   - Tự dựng một cụm với 3 node
   - Kubernetes trên Google Cloud
2. Helm trong Kubernetes
3. Ingress và Networking nâng cao:
   - Ingress Controllers
   - SSL/TLS Certificates
4. Bảo mật Kubernetes:
   - RBAC (Role-Based Access Control)
   - Service Accounts
   - Network Policies
5. Graceful Shutdown trong Kubernetes
6. Quản lý cấu hình:
   - CustomResourceDefinitions (CRDs)
   - Operators

## Phần 3: CI/CD và Triển khai ứng dụng (2-3 tuần)
1. GitLab CI/CD cho Kubernetes
2. Continuous Deployment:
   - ASP.NET Core trên Kubernetes
   - Kubernetes deployment với GitLab CI/CD
3. Chiến lược triển khai:
   - Blue/Green Deployment
   - Canary Releases
   - Rolling Updates

## Phần 4: Observability và Service Mesh (3-4 tuần)
1. Monitoring Kubernetes cluster:
   - Prometheus
   - Grafana
2. Logging trong Kubernetes
3. Tracing phân tán
4. Service Mesh:
   - Istio
   - Envoy proxy
   - Consul Service Mesh
5. Networking and Pod to Pod Communication

## Phần 5: Stateful Applications và Data Services (2-3 tuần)
1. Triển khai Stateful Applications trên Kubernetes
2. Cài đặt và quản lý Database trên Kubernetes
3. Elasticsearch trên Kubernetes
4. Apache Kafka trên Kubernetes

## Phần 6: Bảo mật và Quản lý Secrets (1-2 tuần)
1. Hashicorp Vault trên Kubernetes
2. Secrets Management
3. Encryption at rest và in transit

## Phần 7: Microservices Patterns và Frameworks (2-3 tuần)
1. Dapr (Distributed Application Runtime):
   - Giới thiệu về Dapr
   - Deploy Dapr trên Kubernetes
   - Xây dựng ứng dụng với Dapr
2. API Gateway patterns
3. Circuit Breaker và Retry patterns

## Dự án Tổng hợp (2-3 tuần)
Xây dựng một ứng dụng microservices hoàn chỉnh trên Kubernetes, tích hợp:
- CI/CD pipeline
- Service Mesh (Istio hoặc Consul)
- Monitoring và Logging
- Stateful và Stateless services
- Security practices (Vault, RBAC)
- Sử dụng Dapr cho một số microservices

Tổng thời gian ước tính: 18-26 tuần (4-6 tháng, tùy thuộc vào tốc độ học và thời gian thực hành)
