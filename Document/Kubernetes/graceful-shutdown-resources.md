# Tài liệu học về Graceful Shutdown trong Kubernetes

## Tài liệu chính thức của Kubernetes

1. [Container Lifecycle Hooks](https://kubernetes.io/docs/concepts/containers/container-lifecycle-hooks/)
   - Giải thích về các hooks trong vòng đời của container, bao gồm preStop hook.

2. [Termination of Pods](https://kubernetes.io/docs/concepts/workloads/pods/pod-lifecycle/#pod-termination)
   - Mô tả quá trình kết thúc của một Pod.

3. [Configure Liveness, Readiness and Startup Probes](https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes/)
   - Hướng dẫn cấu hình các probe để kiểm tra sức khỏe của ứng dụng.

4. [Pod Disruption Budgets](https://kubernetes.io/docs/concepts/workloads/pods/disruptions/)
   - Giải thích cách sử dụng PodDisruptionBudgets để duy trì tính khả dụng của ứng dụng.

## Bài viết và hướng dẫn

1. [Kubernetes best practices: terminating with grace](https://cloud.google.com/blog/products/containers-kubernetes/kubernetes-best-practices-terminating-with-grace)
   - Bài viết từ Google Cloud về các best practices cho graceful termination.

2. [Graceful shutdown and zero downtime deployments in Kubernetes](https://learnk8s.io/graceful-shutdown)
   - Hướng dẫn chi tiết về cách triển khai graceful shutdown.

3. [Graceful shutdown of pods with Kubernetes](https://medium.com/flant-com/graceful-shutdown-in-kubernetes-435f595d4f93)
   - Bài viết trên Medium giải thích các khía cạnh khác nhau của graceful shutdown.

## Video tutorials

1. [Kubernetes Pods Termination Lifecycle](https://www.youtube.com/watch?v=Z_l_kE1MDTc)
   - Video giải thích về vòng đời kết thúc của Pods.

2. [Graceful Shutdown with Kubernetes | DevOps and Docker Live Show](https://www.youtube.com/watch?v=Z_GZtm6cMz4)
   - Thảo luận về graceful shutdown trong một live show.

## Tài liệu nâng cao

1. [Kubernetes in Action](https://www.manning.com/books/kubernetes-in-action) (Sách)
   - Chương về Pod lifecycle và container hooks.

2. [Production Kubernetes](https://www.oreilly.com/library/view/production-kubernetes/9781492092292/) (Sách)
   - Có phần thảo luận về high availability và graceful termination.

3. [Kubernetes Patterns](https://www.oreilly.com/library/view/kubernetes-patterns/9781492050278/) (Sách)
   - Giới thiệu các mẫu thiết kế trong Kubernetes, bao gồm cả những mẫu liên quan đến lifecycle management.

## Thực hành

1. [Katacoda Kubernetes Scenarios](https://www.katacoda.com/courses/kubernetes)
   - Cung cấp môi trường thực hành trực tuyến để thử nghiệm các khái niệm Kubernetes.

2. [Kubernetes the Hard Way](https://github.com/kelseyhightower/kubernetes-the-hard-way)
   - Hướng dẫn chi tiết về cách thiết lập một cluster Kubernetes từ đầu, giúp hiểu sâu về cách hoạt động của các thành phần.

