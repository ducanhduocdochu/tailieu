sudo sysctl -w vm.max_map_count=524288
sudo sysctl -w fs.file-max=131072

# Thêm vào file
sudo nano /etc/sysctl.conf
```
vm.max_map_count=524288
fs.file-max=131072
```
sudo sysctl -p

# Cài docker
sudo apt update
sudo apt install docker.io docker-compose -y
sudo systemctl enable docker

mkdir sonarqube
cd sonarqube

nano docker-compose.yml

```
version: "3"

services:

  db:
    image: postgres:13
    container_name: sonar-db
    restart: always
    environment:
      POSTGRES_USER: sonar
      POSTGRES_PASSWORD: sonar
      POSTGRES_DB: sonarqube
    volumes:
      - postgres_data:/var/lib/postgresql/data

  sonarqube:
    image: sonarqube:lts
    container_name: sonarqube
    depends_on:
      - db
    restart: always
    ports:
      - "9000:9000"
    environment:
      SONAR_JDBC_URL: jdbc:postgresql://db:5432/sonarqube
      SONAR_JDBC_USERNAME: sonar
      SONAR_JDBC_PASSWORD: sonar
    volumes:
      - sonar_data:/opt/sonarqube/data
      - sonar_logs:/opt/sonarqube/logs
      - sonar_extensions:/opt/sonarqube/extensions

volumes:
  postgres_data:
  sonar_data:
  sonar_logs:
  sonar_extensions:
```

# chạy
docker-compose up -d

# cài nginx
sudo apt install nginx -y
sudo nano /etc/nginx/sites-available/sonarqube

sudo mkdir /etc/nginx/ssl
sudo nano /etc/nginx/ssl/sonar.crt
sudo nano /etc/nginx/ssl/sonar.key

```
server {
    listen 80;
    server_name sonar.yourdomain.com;

    return 301 https://$host$request_uri;
}

server {
    listen 443 ssl;
    server_name sonar.yourdomain.com;

    ssl_certificate /etc/nginx/ssl/sonar.crt;
    ssl_certificate_key /etc/nginx/ssl/sonar.key;

    location / {
        proxy_pass http://127.0.0.1:9000;

        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-Proto https;
    }
}
```
sudo ln -s /etc/nginx/sites-available/sonarqube /etc/nginx/sites-enabled/
sudo systemctl restart nginx

