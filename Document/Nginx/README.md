# Cấu hình nginx server bằng docker
- Viết cái gì đó ở đây
- 
## Phần 1: Cài đặt nginx:  
 **title:**  
 ```bash
   -- lện ở đây

 ```
## Phần 2: Reverse proxy:  
 **title:**  
 ```bash
   -- lện ở đây

 ```
## Phần 3: Load balancing:  
 **title:**  
 ```bash
   -- lện ở đây

 ```
## Phần 4: Cấu hình HTTP Caching cho Nginx:  
 **title:**  
 ```bash
   -- lện ở đây

 ```

## Phần 5: Cấu hình Rate Limiting :  
 **title:**  
 ```bash
   -- lện ở đây

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
