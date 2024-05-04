# Cấu hình nginx server bằng docker
- NGINX là một web server mạnh mẽ mã nguồn mở. Nginx sử dụng kiến trúc đơn luồng, hướng sự kiện vì thế nó hiệu quả hơn Apache server. Nó cũng có thể làm những thứ quan trọng khác, chẳng hạn như load balancing, HTTP caching, hay sử dụng như một reverse proxy. Nginx là kiến thức không thể thiếu đối với một web developer, system administrator hay devops.


## Phần 1: Cài đặt nginx:  
 **Cài đặt nginx container:**  
 ```bash
   docker run -d -p 80:80 -p 443:443 --restart=always --name nginx  nginx

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

   # Sau khi cấu hình xong
   nginx -s reload
 ```

## Phần 2: Reverse proxy:  
 **Cài đặt Reverse proxy:**  

Chúng ta sửa file cấu hình như sau thì sẽ có Reverse proxy
 
 ```bash
       location /api/creditcard{    
	    proxy_pass http://example.com;
	}

	location /api/debitcard {    
	    proxy_pass http://example.com;
	}
 ```
## Phần 3: Load balancing:  

 **Cấu hình Load balancing:**  
 
 ```bash

	upstream app1 {
	     least_conn;
	     server host1.example.com;    
	     server host2.example.com;    
	     server host3.example.com;
	}
	server {    
	      listen 80;    
	      server_name example.com;
	      location /api/card {       
	        proxy_pass http://app1;   
	    }
	}

 ```

Một số thuật toán Load balancing cơ bản như `Round Robin`,`Least Connections`,`IP Hash`,`Weighted Round Robin` tuỳ trường hợp mà sử dụng.  
## Phần 4: Cấu hình HTTP Caching cho Nginx:  
 **HTTP Caching:**  

-- Chưa lĩnh hội được
 
 ```bash
   -- lệnh ở đây

 ```

## Phần 5: Cấu hình Rate Limiting :  
 **Basic Rate Limiting:**   
   
  Áp dụng giới hạn tốc độ được định nghĩa trong limit_req_zone cho một vị trí cụ thể (/login/ trong ví dụ).  
  Ví dụ, mỗi địa chỉ IP duy nhất bị giới hạn ở 10 yêu cầu mỗi giây đối với URL /login/.  

 
 ```bash

	limit_req_zone $binary_remote_addr zone=mylimit:10m rate=10r/s;
	 
	server {
	    location /login/ {
	        limit_req zone=mylimit;
	        
	        proxy_pass http://my_upstream;
	    }
	}

 ```

 **Handling Bursts algorithm:**  
   
  Định rõ số lượng yêu cầu mà một khách hàng có thể thực hiện vượt quá giới hạn tốc độ đã xác định. Trong ví dụ, burst=20 cho phép một khách hàng gửi đến 20 yêu cầu nhanh 
  chóng trước khi bắt đầu giới hạn.   

Yêu cầu vượt quá giới hạn sẽ được đưa vào hàng đợi và được xử lý đúng cách.   
 
 ```bash

location /login/ {
    limit_req zone=mylimit burst=20;
 
    proxy_pass http://my_upstream;
}
 ```

 **Queueing with No Delay algorithm:**
  
  Cùng với burst, thêm nodelay giúp giữ cho lưu lượng đi qua mà không làm trang web trở nên chậm chạp.
  Ngay khi có khe trống trong hàng đợi, yêu cầu vượt quá giới hạn sẽ được chuyển tiếp ngay lập tức.

 
 ```bash

	location /login/ {
	    limit_req zone=mylimit burst=20 nodelay;
	 
	    proxy_pass http://my_upstream;
	}

 ```

 **Two-Stage Rate Limiting algorithm:**

  Kỹ thuật này (kích hoạt bằng delay parameter) cho phép một sự linh hoạt hai giai đoạn, giúp xử lý những lượng yêu cầu đột ngột và duy trì một giới hạn tốc độ dài hạn.
  Trong ví dụ, 8 yêu cầu đầu tiên được xử lý mà không có độ trễ, nhưng sau đó, những yêu cầu vượt quá giới hạn sẽ được trì hoãn để duy trì tốc độ 5 yêu cầu mỗi giây.
  
 
 ```bash

	limit_req_zone $binary_remote_addr zone=ip:10m rate=5r/s;

	server {
	    listen 80;
	    location / {
	        limit_req zone=ip burst=12 delay=8;
	        proxy_pass http://website;
	    }
	}


 ```

Tham khảo thêm ở https://www.nginx.com/blog/rate-limiting-nginx/

## Phần 6: Cấu hình reverse proxy, load Balance và Let's Encrypt SSL cho container:  
 **Cài đặt nginx proxy:**  

 ```bash
    docker run -d -p 80:80 -p 443:443 --name nginx-proxy --restart=always --privileged=true \
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
   docker run -d --restart=always --privileged=true \
	-v ~/nginx/vhost.d:/etc/nginx/vhost.d \
	-v ~/nginx-certs:/etc/nginx/certs:rw \
	-v /var/run/docker.sock:/var/run/docker.sock:ro \
	--volumes-from nginx-proxy \
	jrcs/letsencrypt-nginx-proxy-companion
 ```
Chú ý: volumes-from cho đúng với server nginx

**Chạy web container lên nhớ gắn các thông số chứng chỉ SSL:**  
 ```bash

# VIRTUAL_PORT là port của website
# ví dụ http://35.222.111.241:5004 thì để
# VIRTUAL_HOST không để http hoặc https

   docker run -it -d --name containerName --restart=always \
	-e VIRTUAL_HOST="your-domain.vn" \
	-e VIRTUAL_PORT=5004 \
	-e LETSENCRYPT_HOST="your-domain.vn" \
	-e LETSENCRYPT_EMAIL="vanhungdev@fpt.com.vn" \
	vanhungdev/imageName:v.1.1

 ```

Lưu ý: nếu muốn chạy 2 host độc lập thì chạy lại server api giống nhau đổi VIRTUAL_HOST và containerName.

**Cấu hình cho web số 2:**  

 ```bash
   docker run -it -d --name containerName-2 --restart=always \
	-e VIRTUAL_HOST="your-domain-2.vn" \
	-e VIRTUAL_PORT=80 \
	-e LETSENCRYPT_HOST="your-domain-2.vn" \
	-e LETSENCRYPT_EMAIL="vanhungdev@fpt.com.vn" \
	vanhungdev/imageName:v.1.1

 ```
