## E20241211 docker group.md 
- Docker가 sudo에서만 돌 때

### Create the docker group if it does not exist:
$ sudo groupadd docker

### Add your user to the docker group:
$ sudo usermod -aG docker $USER

### Log in to the new docker group (to avoid having to log out and log in again; but if not enough, try to reboot):
$ newgrp docker
