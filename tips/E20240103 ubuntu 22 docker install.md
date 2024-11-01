## E20240103 ubuntu 22 docker install.md

```
#!/bin/bash

# Docker 패키지를 업데이트하고 필수 패키지를 설치합니다
sudo apt-get update
sudo apt-get install -y \
    apt-transport-https \
    ca-certificates \
    curl \
    software-properties-common

# Docker의 공식 GPG 키를 추가합니다
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg

# Docker Stable 저장소를 추가합니다
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# Docker 패키지를 업데이트하고 Docker 엔진을 설치합니다
sudo apt-get update
sudo apt-get install -y docker-ce docker-ce-cli containerd.io

# Docker 설치가 완료되었는지 확인합니다
sudo docker --version

# (옵션) 현재 사용자를 Docker 그룹에 추가하여 sudo 없이 Docker 명령을 사용할 수 있게 합니다
sudo usermod -aG docker $USER

echo "Docker 설치가 완료되었습니다. Docker 버전을 확인하려면 'docker --version' 명령을 사용하세요."

# 변경 사항 적용을 위해 로그아웃 후 다시 로그인하거나, 다음 명령을 실행하여 그룹 변경을 적용할 수 있습니다.
newgrp docker
```
