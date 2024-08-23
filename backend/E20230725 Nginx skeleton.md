##  Nginx skeleton

### Dockerfile
```
FROM nginx:latest
COPY ./nginx.conf                /etc/nginx/nginx.conf
COPY ./mymarket-product.conf     /etc/nginx/conf.d/mymarket-product.conf

COPY ./css                      /usr/share/nginx/mymarket/product/css
COPY ./html                     /usr/share/nginx/mymarket/product/html
COPY ./js                       /usr/share/nginx/mymarket/product/js
COPY ./favicon.ico              /usr/share/nginx/mymarket/product/
CMD ["nginx", "-g", "daemon off;"]

EXPOSE 80
```

### /etc/nginx/nginx.conf

```
worker_processes  1;
error_log  /var/log/nginx/error.log warn;
pid        /var/run/nginx.pid;

events {
    worker_connections  1024;
}

http {
    include       /etc/nginx/mime.types;
    default_type  application/octet-stream;

    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    access_log         /var/log/nginx/access.log  main;
    sendfile           on;
    keepalive_timeout  65;

    include              /etc/nginx/conf.d/mymarket-product.conf;
}
```

### /etc/nginx/conf.d/mymarket-product.conf
```
server {
    listen       80;
    server_name  localhost;

    # root location
    location / {
       root   /usr/share/nginx/mymarket/product;
       index  html/product.html;
    }

    # server error
    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   /usr/share/nginx/html;
    }
}
```

### root 와 alias 차이

find file from '/usr/share/nginx/html/test/{file}'
```
location /test-root/ {
   root    /usr/share/nginx/html;
}
```

find file from '/usr/share/nginx/html/{file}'
```
location /test-alias/ {
   alias   /usr/share/nginx/html;
}
```
