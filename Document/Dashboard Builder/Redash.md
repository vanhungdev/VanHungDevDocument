# Redash là gì?

Redash là một nền tảng nguồn mở mạnh mẽ giúp bạn khám phá, truy vấn, trực quan hóa và chia sẻ dữ liệu từ nhiều nguồn khác nhau. Đây là một công cụ tuyệt vời cho các nhà phân tích dữ liệu, kỹ sư dữ liệu và các nhóm kinh doanh muốn khai thác thông tin chi tiết từ dữ liệu của mình.

**Các tính năng chính:**

*   **Kết nối đa dạng:** Redash hỗ trợ kết nối với nhiều nguồn dữ liệu phổ biến như PostgreSQL, MySQL, SQLite, Google BigQuery, Amazon Redshift, Snowflake, và nhiều hơn nữa.
   
*   **Truy vấn linh hoạt:** Bạn có thể viết các truy vấn SQL hoặc sử dụng giao diện trực quan để tạo các truy vấn phức tạp.
   
*   **Trực quan hóa đa dạng:** Tạo các biểu đồ, bảng, bản đồ và các hình thức trực quan hóa khác để khám phá dữ liệu một cách trực quan.
   
*   **Chia sẻ và cộng tác:** Dễ dàng chia sẻ các dashboard và báo cáo với đồng nghiệp, tạo điều kiện cho sự cộng tác và ra quyết định dựa trên dữ liệu.
   
*   **Tùy chỉnh:** Redash có thể được mở rộng và tùy chỉnh để đáp ứng các yêu cầu cụ thể của bạn.
   

# Cài đặt và xây dựng hệ thống Redash

Có nhiều cách để cài đặt Redash, từ việc sử dụng Docker cho đến cài đặt thủ công trên máy chủ của bạn. Dưới đây là một hướng dẫn chung về cách cài đặt Redash bằng Docker Compose (một cách phổ biến và dễ dàng):

**1\. Cài đặt Docker và Docker Compose:**

Đảm bảo bạn đã cài đặt Docker và Docker Compose trên hệ thống của mình. Bạn có thể tìm thấy hướng dẫn cài đặt chi tiết trên trang web của Docker.

**2\. Tạo file** `docker-compose.yml`:

Tạo một file `docker-compose.yml` với nội dung sau:

```yaml
version: "3"
services:
  server:
    image: redash/redash:preview
    depends_on:
      init-server:
        condition: service_completed_successfully
    restart: always
    environment:
      PYTHONUNBUFFERED: "0"
      REDASH_LOG_LEVEL: "INFO"
      REDASH_REDIS_URL: "redis://redis:6379/0"
      REDASH_COOKIE_SECRET: "xuiCIAEfejj05ZUzQ0odb5pwBttXUkE4"
      REDASH_SECRET_KEY: "9emtXqroCnNhFiETBiFXoQvA4PH0nQuP"
      REDASH_DATABASE_URL: "postgresql://postgres:YKK1KkvPb4ula2HRXlnd7z7n6rVbWKSq@postgres/postgres"
      REDASH_RATELIMIT_ENABLED: "false"
      REDASH_MAIL_DEFAULT_SENDER: "redash@example.com"
      REDASH_MAIL_SERVER: "email"
      REDASH_MAIL_PORT: 1025
      REDASH_ENFORCE_CSRF: "true"
      REDASH_GUNICORN_TIMEOUT: 60
    ports:
      - "5000:5000"
    platform: linux/amd64
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:5000/ping"]
      interval: 1s
      timeout: 2s
      retries: 30


  init-server:
    image: redash/redash:preview
    depends_on:
      - postgres
      - redis
    command:
      - create_db
    environment:
      PYTHONUNBUFFERED: "0"
      REDASH_LOG_LEVEL: "INFO"
      REDASH_REDIS_URL: "redis://redis:6379/0"
      REDASH_COOKIE_SECRET: "xuiCIAEfejj05ZUzQ0odb5pwBttXUkE4"
      REDASH_SECRET_KEY: "9emtXqroCnNhFiETBiFXoQvA4PH0nQuP"
      REDASH_DATABASE_URL: "postgresql://postgres:YKK1KkvPb4ula2HRXlnd7z7n6rVbWKSq@postgres/postgres"
      REDASH_RATELIMIT_ENABLED: "false"
      REDASH_MAIL_DEFAULT_SENDER: "redash@example.com"
      REDASH_MAIL_SERVER: "email"
      REDASH_MAIL_PORT: 1025
      REDASH_ENFORCE_CSRF: "true"
      REDASH_GUNICORN_TIMEOUT: 60
    ports:
      - "5000:5000"
    platform: linux/amd64

  scheduler:
    image: redash/redash:preview
    depends_on:
      - redis
    restart: always
    command: scheduler
    environment:
      QUEUES: "celery"
      WORKERS_COUNT: 1
      PYTHONUNBUFFERED: "0"
      REDASH_LOG_LEVEL: "INFO"
      REDASH_REDIS_URL: "redis://redis:6379/0"
      REDASH_COOKIE_SECRET: "xuiCIAEfejj05ZUzQ0odb5pwBttXUkE4"
      REDASH_SECRET_KEY: "9emtXqroCnNhFiETBiFXoQvA4PH0nQuP"
      REDASH_DATABASE_URL: "postgresql://postgres:YKK1KkvPb4ula2HRXlnd7z7n6rVbWKSq@postgres/postgres"
      REDASH_RATELIMIT_ENABLED: "false"
      REDASH_MAIL_DEFAULT_SENDER: "redash@example.com"
      REDASH_MAIL_SERVER: "email"
      REDASH_MAIL_PORT: 1025
      REDASH_ENFORCE_CSRF: "true"
      REDASH_GUNICORN_TIMEOUT: 60
    platform: linux/amd64

  worker:
    image: redash/redash:preview
    depends_on:
      - redis
    restart: always
    command: worker
    environment:
      QUEUES: "scheduled_queries,schemas,periodic,emails,default"
      WORKERS_COUNT: 6
      PYTHONUNBUFFERED: "0"
      REDASH_LOG_LEVEL: "INFO"
      REDASH_REDIS_URL: "redis://redis:6379/0"
      REDASH_COOKIE_SECRET: "xuiCIAEfejj05ZUzQ0odb5pwBttXUkE4"
      REDASH_SECRET_KEY: "9emtXqroCnNhFiETBiFXoQvA4PH0nQuP"
      REDASH_DATABASE_URL: "postgresql://postgres:YKK1KkvPb4ula2HRXlnd7z7n6rVbWKSq@postgres/postgres"
      REDASH_RATELIMIT_ENABLED: "false"
      REDASH_MAIL_DEFAULT_SENDER: "redash@example.com"
      REDASH_MAIL_SERVER: "email"
      REDASH_MAIL_PORT: 1025
      REDASH_ENFORCE_CSRF: "true"
      REDASH_GUNICORN_TIMEOUT: 60
    platform: linux/amd64

  adhoc-worker:
    image: redash/redash:preview
    depends_on:
      - redis
    restart: always
    command: worker
    environment:
      QUEUES: "queries"
      WORKERS_COUNT: 2
      PYTHONUNBUFFERED: "0"
      REDASH_LOG_LEVEL: "INFO"
      REDASH_REDIS_URL: "redis://redis:6379/0"
      REDASH_COOKIE_SECRET: "xuiCIAEfejj05ZUzQ0odb5pwBttXUkE4"
      REDASH_SECRET_KEY: "9emtXqroCnNhFiETBiFXoQvA4PH0nQuP"
      REDASH_DATABASE_URL: "postgresql://postgres:YKK1KkvPb4ula2HRXlnd7z7n6rVbWKSq@postgres/postgres"
      REDASH_RATELIMIT_ENABLED: "false"
      REDASH_MAIL_DEFAULT_SENDER: "redash@example.com"
      REDASH_MAIL_SERVER: "email"
      REDASH_MAIL_PORT: 1025
      REDASH_ENFORCE_CSRF: "true"
      REDASH_GUNICORN_TIMEOUT: 60
    platform: linux/amd64

  redis:
    image: redis:7.0-alpine
    restart: always

  postgres:
    image: postgres:15-alpine
    restart: always
    environment:
      POSTGRES_PASSWORD: "YKK1KkvPb4ula2HRXlnd7z7n6rVbWKSq"

  nginx:
    image: redash/nginx:latest
    ports:
      - "80:80"
    depends_on:
      - server
    links:
      - server:redash
    restart: always

  email:
    image: maildev/maildev
    ports:
    - "1080:1080"
    - "1025:1025"
    restart: always

```

**3\. Chạy Docker Compose:**

Mở terminal, di chuyển đến thư mục chứa file `docker-compose.yml` và chạy lệnh:

```bash
docker-compose up -d
```

**4 Truy cập Redash:**  
Mở trình duyệt và truy cập địa chỉ http://localhost:5000. Bạn sẽ thấy trang đăng nhập của Redash.


**5 Tạo tài khoản và bắt đầu sử dụng:**  
Tạo một tài khoản quản trị viên và bắt đầu khám phá Redash. Bạn có thể kết nối với các nguồn dữ liệu, viết truy vấn, tạo dashboard và chia sẻ chúng với đồng nghiệp.