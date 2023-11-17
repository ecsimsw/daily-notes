### Server - add user
```
sudo adduser ecsimsw
sudo adduser ecsimsw sudo
sudo chown -R ecsimsw /home/
```

### Client - rsa 
```
ls -l ~/.ssh/id_*.pub

ssh-keygen -t rsa -b 4096 -C "ecsimsw@ecsimsw.com"

ls ~/.ssh/id_*
```

### Server - rsa pub
```
cat ~/.ssh/authorized_keys

sudo vi ~/.ssh/authorized_keys

chmod 600 ~/.ssh/authorized_keys
```
