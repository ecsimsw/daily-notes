## E20241101 proxy 서버 ngrep.md

- 모바일 SDK -> 외부 서버로 전송되는 이벤트를 서버 측에서 기록해야 하는 요구사항이 있었다.
- 프록시 서버로 Nginx를 설정하고 이벤트 꼴 확인을 위해 Logging을 하고 있었는데
- ngrep으로 그냥 프록시 서버를 잡아 패킷 확인을 했으면 됐던걸 싶다.
- 만들어둔 nginx는 포맷 예쁘게 해서 로그 기록용으로 하고, 원래 필요했던 패킷 캡쳐는 nginx 포트를 ngrep으로 잡는걸로!!!

### ngrep command
``` 
sudo ngrep -q -W byline port 8080
```


### nginx.conf
``` yaml

events {
    worker_connections 1024;
}

http {
    server {
        listen 80;
        listen [::]:80;

       ...
       
       lua_need_request_body on;
       set $resp_body "";
       body_filter_by_lua '
            local resp_body = string.sub(ngx.arg[1], 1, 1000)
            ngx.ctx.buffered = (ngx.ctx.buffered or "") .. resp_body
            if ngx.arg[2] then
                ngx.var.resp_body = ngx.ctx.buffered
            end
       ';
    }

    log_format log_req_resp '$remote_addr - $remote_user [$time_local] '
        '"$request" $req_body $status $body_bytes_sent '
        '"$http_referer" "$http_user_agent" $request_time req_body:"$request_body" resp_body:"$resp_body"';

    access_log  /var/log/nginx/access.log  log_req_resp;
}
```

### dockerfile
``` yaml
services:
  nginx:
    image: nginx:1.21.5-alpine
    ports:
      - "8080:80"
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf
      - ./index.html:/etc/nginx/html/index.html
      - ./log:/var/log/nginx
    container_name: nginx-notification
```
