# E20241101 Ubuntu user init.md

- 개발팀 Ubuntu 세팅 중

### New user
```
# 유저 생성
sudo adduser <새로운_사용자_이름>

# sudo 권한 부여
sudo usermod -aG sudo <새로운_사용자_이름>
```

### ssh key
```
# ssh 키 복사
sudo cp -r /home/ubuntu/.ssh /home/<새로운_사용자_이름>/
sudo chown -R <새로운_사용자_이름>:<새로운_사용자_이름> /home/<새로운_사용자_이름>/.ssh
```

### ssh config
```
# ssh config 확인
sudo vi /etc/ssh/sshd_config

#### config start

# ssh 허용 사용자 추가
AllowUsers <새로운_사용자_이름>

# password 접속 허용
PasswordAuthentication yes

#### config end

# 설정 반영
sudo systemctl restart ssh
```


