## E20240104 install nginx.md

#### install
```
sudo apt update
sudo apt install nginx
```

#### UFW
```
sudo ufw app list
sudo ufw allow 'Nginx HTTP'
sudo ufw status
```

#### control
```
sudo systemctl start nginx

sudo systemctl stop nginx

sudo systemctl restart nginx

sudo systemctl reload nginx
```

#### remove all
```
apt-get remove --purge nginx nginx-full nginx-common
```
