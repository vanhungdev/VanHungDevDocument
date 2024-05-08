## Prometheus và Grafana: Hệ thống giám sát linux

**Prometheus và Grafana** là hai công cụ mã nguồn mở được sử dụng rộng rãi để giám sát hiệu suất hệ thống và ứng dụng. Chúng hoạt động cùng nhau để cung cấp cho bạn một cái nhìn toàn diện về hoạt động của hệ thống của bạn.

### Prometheus là gì?

**Prometheus** là một hệ thống thu thập dữ liệu và giám sát theo thời gian thực. Nó hoạt động bằng cách thu thập các "metric" (chỉ số) từ các mục tiêu được định cấu hình (chẳng hạn như máy chủ, ứng dụng, container). Các metric này có thể bao gồm mức sử dụng CPU, bộ nhớ, lưu lượng truy cập mạng, thời gian phản hồi API, v.v. Prometheus lưu trữ dữ liệu metric trong một cơ sở dữ liệu thời gian (time series database).

### Grafana là gì?

**Grafana** là một bảng điều khiển web cho phép bạn trực quan hóa dữ liệu metric được thu thập bởi Prometheus. Nó cung cấp nhiều loại biểu đồ, bảng, bảng điều khiển và các công cụ trực quan khác để giúp bạn dễ dàng hiểu và phân tích dữ liệu giám sát của mình. Grafana cũng cho phép bạn tạo các cảnh báo để được thông báo khi có sự cố xảy ra trong hệ thống của bạn.

### Cách Prometheus và Grafana hoạt động cùng nhau:

1. **Prometheus thu thập dữ liệu:** Prometheus truy vấn các mục tiêu được định cấu hình để thu thập các metric.
2. **Prometheus lưu trữ dữ liệu:** Prometheus lưu trữ dữ liệu metric trong cơ sở dữ liệu thời gian.
3. **Grafana kết nối với Prometheus:** Grafana kết nối với Prometheus để truy cập dữ liệu metric.
4. **Grafana hiển thị dữ liệu:** Grafana sử dụng dữ liệu metric để tạo các biểu đồ, bảng, bảng điều khiển và các trực quan hóa khác.
5. **Grafana tạo cảnh báo:** Grafana có thể tạo cảnh báo khi phát hiện các ngưỡng được xác định trước bị vi phạm.

### Lợi ích của việc sử dụng Prometheus và Grafana:

- **Giám sát hiệu suất hệ thống:** Prometheus và Grafana cung cấp cho bạn cái nhìn toàn diện về hiệu suất hệ thống của bạn, giúp bạn xác định các vấn đề tiềm ẩn và tối ưu hóa hiệu suất.
- **Khả năng hiển thị:** Grafana cung cấp các trực quan hóa mạnh mẽ giúp bạn dễ dàng hiểu dữ liệu giám sát của mình.
- **Cảnh báo:** Grafana có thể tạo cảnh báo để thông báo cho bạn khi có sự cố xảy ra trong hệ thống của bạn.
- **Mã nguồn mở:** Prometheus và Grafana đều là mã nguồn mở, có nghĩa là bạn có thể sử dụng chúng miễn phí và tùy chỉnh theo nhu cầu của mình.

**Prometheus** cũng có thể được sử dụng để thu thập dữ liệu nhật ký và sự kiện. **Grafana** có thể kết nối với nhiều nguồn dữ liệu khác ngoài Prometheus, chẳng hạn như Graphite, InfluxDB và Elasticsearch. Có nhiều plugin và công cụ bổ sung có sẵn cho Prometheus và Grafana để mở rộng chức năng của chúng.



### Cài đặt:
Cài nano nếu chưa có

```bash

sudo yum install nano

Ctrl + O (viết tắt của "Output"): Dùng để lưu file.

enter

Ctrl + X (viết tắt của "eXit"): Thoát khỏi nano.

```


Tạo thư mục chưa file config Prometheus

```bash

mkdir -p /etc/prometheus

nano prometheus.yml

```

Thông tin cấu hình prometheus
Bỏ đoạn code này vào file `prometheus.yml` sau đó lưu và thoát

```bash
global:
  scrape_interval:     15s # By default, scrape targets every 15 seconds.

  # Attach these labels to any time series or alerts when communicating with
  # external systems (federation, remote storage, Alertmanager).
  # external_labels:
  #  monitor: 'codelab-monitor'

# A scrape configuration containing exactly one endpoint to scrape:
# Here it's Prometheus itself.
scrape_configs:
  # The job name is added as a label `job=<job_name>` to any timeseries scraped from this config.
  - job_name: 'prometheus'
    # Override the global default and scrape targets from this job every 5 seconds.
    scrape_interval: 5s
    static_configs:
      - targets: ['localhost:9090']

  # Example job for node_exporter
  - job_name: 'node_exporter'
    static_configs:
      - targets: ['node_exporter:9100']
#   #intanse-01       
#   - job_name: 'node_exporter_intance01'
#     static_configs:
#       - targets: ['188.166.222.232:9100']
      
  # Example job for cadvisor
  - job_name: 'cadvisor'
    static_configs:
      - targets: ['cadvisor:8080']

```

Đưa file config vào thư mục

```bash
cp prometheus.yml /etc/prometheus
```

**Tạo docker file để chạy container**

```bash 

nano docker-compose.yml


```

Bỏ nội dung này vào

```bash
---
version: '3'

volumes:
  prometheus-data:
    driver: local
  grafana-data:
    driver: local

services:
  prometheus:
    image: prom/prometheus:latest
    container_name: prometheus
    ports:
      - "9090:9090"
    volumes:
      - /etc/prometheus:/etc/prometheus
      - prometheus-data:/prometheus
    restart: unless-stopped
    command:
      - "--config.file=/etc/prometheus/prometheus.yml"

  grafana:
    image: grafana/grafana:latest
    container_name: grafana
    ports:
      - "3000:3000"
    volumes:
      - grafana-data:/var/lib/grafana
    environment:
      - 'GF_SMTP_ENABLED=true'
      - 'GF_SMTP_HOST=smtp.gmail.com:587'
      - 'GF_SMTP_USER=user1@gmail.com'
      - 'GF_SMTP_PASSWORD=mysamplePassword'
      - 'GF_SMTP_FROM_ADDRESS=user1@gmail.com'  
      #- 'GF_SERVER_DOMAIN=grafana.my.domain'
      #- 'GF_SERVER_ROOT_URL=grafana.my.domain'
    restart: unless-stopped
  node_exporter:
    image: quay.io/prometheus/node-exporter:latest
    container_name: node_exporter
    command:
      - '--path.rootfs=/host'
    pid: host
    restart: unless-stopped
    volumes:
      - '/:/host:ro,rslave'
  cadvisor:
    image: google/cadvisor:latest
    container_name: cadvisor
    # ports:
    #   - "8080:8080"
    volumes:
      - /:/rootfs:ro
      - /var/run:/var/run:ro
      - /sys:/sys:ro
      - /var/lib/docker/:/var/lib/docker:ro
      - /dev/disk/:/dev/disk:ro
    devices:
      - /dev/kmsg

```


Sau đó chạy lệnh

```bash
docker-compose  up -d

#hoặc docker compose  up -d

docker-compose  ps


```

sau khi chạy xong sẽ thấy 4 container 
 `cadvitor`, `grafana`, `node_exporter`, `prometheus`

Sẽ chạy port 3000
```bash
http://yor-ip:3000 
```

