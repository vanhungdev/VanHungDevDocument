# Cấu hình jenkins
- Jenkins

```bash

https://github.com/jenkinsci/docker

// Chạy container
docker run -p 8080:8080 -p 50000:50000 --restart=on-failure -v jenkins_home:/var/jenkins_home jenkins/jenkins:lts-jdk17

docker run -p 8080:8080 -p 50000:50000 --restart=on-failure -v jenkins_home:/var/jenkins_home --network 	jenkins jenkins/jenkins:lts-jdk17

docker run -d --restart=always -p 127.0.0.1:2376:2375 --network jenkins -v /var/run/docker.sock:/var/run/docker.sock alpine/socat tcp-listen:2375,fork,reuseaddr unix-connect:/var/run/docker.sock


// Lấy password
docker exec trusting_gagarin cat /var/jenkins_home/secrets/initialAdminPassword
```
