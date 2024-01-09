# Cấu hình Elasticsearch và kibana
- Viết cái gì đó ở đây

## Phần 1: Cài đặt ELK Stack:  

**Để cài được Elasticsearch, kibna chúng ta cần cài theo các bước sau:**   

1. **Tạo elastic network** - Cần thiết để kết nối elasticsearch với kibna.
2. **Tạo elastic-server container** - Chúng ta sẽ chạy container elasticsearch trước.
3. **Lấy password của tài khoản kibana_system** - Cần lấy password của tk kibana_system vì kibana sẽ không kết nối với tk elastic.
4. **Chạy kibna container** - Chúng ta cần chạy kiban container.
5. **Lấy code từ log kibana và kết nối với elasticsearch** - Chạy thử ở môi trường thật.

**Tạo elastic network:**  

 ```bash
  docker network create elastic
 ```
**Bỏ limit size của elasticsearch server:**  
 
 ```bash
sudo vim /etc/sysctl.conf
 ```
Chèn dòng cấu hình `vm.max_map_count=262144` bằng cách nhập i để chuyển vào chế độ chèn, sau đó nhập dòng cấu hình.  

Sau khi cài xong chạy lệnh
 
 ```bash
sudo sysctl -w vm.max_map_count=262144
 ```

**Cấu hình và tạo elastic-server container:**  
  Tạo elastic-server container:  
 
 ```bash
  docker run --name elastic-server --net elastic -p 9200:9200 -p 9300:9300 -e "discovery.type=single-node" -e "network.host=0.0.0.0"  -e "xpack.security.enabled=true" -e "ELASTIC_PASSWORD=Provanhung77" -e "xpack.security.http.ssl.enabled=false" -t docker.elastic.co/elasticsearch/elasticsearch:8.11.3

 ```

  Lấy password của tài khoản kibana_system:  
 
 ```bash
docker exec -it elastic-server /usr/share/elasticsearch/bin/elasticsearch-reset-password -u kibana_system

 ```
 Nếu vào exec vào rồi hoặc dùng portainer thì chỉ cần 

  ```bash
/usr/share/elasticsearch/bin/elasticsearch-reset-password -u kibana_system

 ```

**Chạy kibna container:**  
 
 ```bash
  docker run --name kib01 --net elastic -p 5601:5601 docker.elastic.co/kibana/kibana:8.11.3
 ```
Lưu ý: cần xem log của kibana và lấy code  
elastic_server là http://elastic-server:9200 hoặc là ip của container name của elasticsearch là được

## Phần 2: Tạo và đánh index elastic: 
Viết cái gì đó ở đây

## Phần 3: Một số câu query elastic cơ bản: 
Viết cái gì đó ở đây
