# Cấu hình jenkins
- Jenkins

```bash

https://github.com/jenkinsci/docker

// Chạy container
docker run -p 8080:8080 -p 50000:50000 --restart=on-failure -v jenkins_home:/var/jenkins_home jenkins/jenkins:lts-jdk17


// Lấy password
docker exec trusting_gagarin cat /var/jenkins_home/secrets/initialAdminPassword
```
