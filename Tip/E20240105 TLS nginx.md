## E20240105 TLS nginx.md

```
sudo apt install certbot python3-certbot-nginx
sudo certbot certonly --standalone -d anime-kr.ecsimsw.com
```

```
server {
    ssl_certificate /etc/letsencrypt/live/anime-kr.ecsimsw.com/fullchain.pem;
	  ssl_certificate_key /etc/letsencrypt/live/anime-kr.ecsimsw.com/privkey.pem;
}
```
