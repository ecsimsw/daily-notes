## E20240105 TLS nginx.md

```
sudo apt install certbot python3-certbot-nginx
sudo certbot certonly --standalone -d anime-kr.ecsimsw.com
```

#### If 80 port is already used by dummy
```
sudo lsof -t -i tcp:80 -s tcp:listen | sudo xargs kill
```

#### Nginx
```
server {
    ssl_certificate /etc/letsencrypt/live/anime-kr.ecsimsw.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/anime-kr.ecsimsw.com/privkey.pem;
}
```

#### sample - nginx.conf

```
worker_processes  1;
error_log  /var/log/nginx/error.log warn;
pid        /var/run/nginx.pid;

events {
    worker_connections  1024;
}

http {
    limit_req_zone $binary_remote_addr zone=default_rate_limit:10m rate=5r/s;

    upstream api-service {
        server host.docker.internal:8080;
    }

    server {
      listen 80;
      return 301 https://anime-kr.ecsimsw.com$request_uri;
    }

    server {
      listen 443 ssl;
      server_name anime-kr.ecsimsw.com;

      ssl_certificate /etc/letsencrypt/live/anime-kr.ecsimsw.com/fullchain.pem;
      ssl_certificate_key /etc/letsencrypt/live/anime-kr.ecsimsw.com/privkey.pem;

      location / {
        proxy_pass http://api-service;
        proxy_set_header Host $http_host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;

        limit_req zone=default_rate_limit burst=5 nodelay;
        limit_req_status 429;
      }
    }

    include       /etc/nginx/mime.types;
    default_type  application/octet-stream;

    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    access_log         /var/log/nginx/access.log  main;
    sendfile           on;
    keepalive_timeout  65;
}
```

#### sample - docker-compose
```
version: '3'
services:
  anime-kr-gateway:
    image: nginx
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf
      - /etc/letsencrypt:/etc/letsencrypt
    ports:
      - "80:80"
      - "443:443"
    extra_hosts:
      - "host.docker.internal:host-gateway"
```

