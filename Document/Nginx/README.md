# Cấu hình nginx server bằng docker
- NGINX là một web server mạnh mẽ mã nguồn mở. Nginx sử dụng kiến trúc đơn luồng, hướng sự kiện vì thế nó hiệu quả hơn Apache server. Nó cũng có thể làm những thứ quan trọng khác, chẳng hạn như load balancing, HTTP caching, hay sử dụng như một reverse proxy. Nginx là kiến thức không thể thiếu đối với một web developer, system administrator hay devops.


## Phần 1: Cài đặt nginx:  
 **Cài đặt nginx container:**  
 ```bash
   docker run -d -p 80:80 -p 443:443 --name nginx nginx

 ```

Cài đặt vi hoặc nano để chỉnh sửa file
```bash
   apt-get update
   apt-get install -y vim
   apt-get install -y nano
 ```


Truy cập vào file config
```bash
   docker exec -it nginx /bin/bash

   nano /etc/nginx/conf.d/default.conf
   vi /etc/nginx/conf.d/default.conf

 ```

## Phần 2: Reverse proxy:  
 **title:**  
 ```bash
   -- lệnh ở đây

 ```
## Phần 3: Load balancing:  
 **title:**  
 ```bash
   -- lệnh ở đây

 ```
## Phần 4: Cấu hình HTTP Caching cho Nginx:  
 **title:**  
 ```bash
   -- lệnh ở đây

 ```

## Phần 5: Cấu hình Rate Limiting :  
 **title:**  
 ```bash
   -- lệnh ở đây

 ```


## Phần 6: Cấu hình reverse proxy, load Balance và Let's Encrypt SSL cho container:  
 **Cài đặt nginx proxy:**  

 ```bash
    docker run -d -p 80:80 -p 443:443 --name nginx-proxy --privileged=true \
	-e ENABLE_IPV6=true \
	-v ~/nginx/vhost.d:/etc/nginx/vhost.d \
	-v ~/nginx-certs:/etc/nginx/certs:ro \
	-v ~/nginx-conf:/etc/nginx/conf.d \
	-v ~/nginx-logs:/var/log/nginx \
	-v /usr/share/nginx/html \
	-v /var/run/docker.sock:/tmp/docker.sock:ro \
	jwilder/nginx-proxy
 ```
 **Tải chứng chỉ Let's Encrypt SSL cho nginx:**  
  

 ```bash
   docker run -d --privileged=true \
	-v ~/nginx/vhost.d:/etc/nginx/vhost.d \
	-v ~/nginx-certs:/etc/nginx/certs:rw \
	-v /var/run/docker.sock:/var/run/docker.sock:ro \
	--volumes-from nginx-proxy \
	jrcs/letsencrypt-nginx-proxy-companion
 ```
Chú ý: volumes-from cho đúng với server nginx

**Chạy web container lên nhớ gắn các thông số chứng chỉ SSL:**  
 ```bash
   docker run -it -d --name containerName \
	-e VIRTUAL_HOST="your-domain.vn" \
	-e VIRTUAL_PORT=80 \
	-e LETSENCRYPT_HOST="your-domain.vn" \
	-e LETSENCRYPT_EMAIL="vanhungdev@fpt.com.vn" \
	vanhungdev/imageName:v.1.1

 ```

Lưu ý: nếu muốn chạy 2 host độc lập thì chạy lại server api giống nhau đổi VIRTUAL_HOST và containerName.

**Cấu hình cho web số 2:**  

 ```bash
   docker run -it -d --name containerName-2 \
	-e VIRTUAL_HOST="your-domain-2.vn" \
	-e VIRTUAL_PORT=80 \
	-e LETSENCRYPT_HOST="your-domain-2.vn" \
	-e LETSENCRYPT_EMAIL="vanhungdev@fpt.com.vn" \
	vanhungdev/imageName:v.1.1

 ```

**Cấu hình reverse proxy:**  
- Đang nguyên cứu

**Load balancer:**  
- Đang nguyên cứu
