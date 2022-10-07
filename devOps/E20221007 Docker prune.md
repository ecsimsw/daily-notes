# Docker의 prune
Prune : 잘라내다, 간결하게 하다, 정리하다.  
Docker의 prune 사용법 정리하기  

#### docker container prune

도커 컨테이너도 결국 리눅스 프로세스다.     
중지된 컨테이너는 메모리 자원을 사용하진 않지만, 각 컨테이너는 고유한 디스크 영역을 갖고 있어 제거하는데 의미를 갖는다.

```
# 중지된 모든 컨테이너 삭제
$ docker container prune

# 중지된 지 1시간 이상 지난 컨테이너 삭제
$ docker container prune --filter until=1h

# 라벨이 존재하는 컨테이너 삭제
$ docker container prune --filter label=env

# 라벨이 존재하지 않는 컨테이너 삭제
$ docker container prune --filter label!=env

# 라벨의 값을 확인하여 컨테이너 삭제
$ docker container prune --filter label=env=development

# 라벨의 값을 확인하여 다르다면 컨테이너 삭제
$ docker container prune --filter label!=env=production
```

#### docker image prune

도커 이미지는 이름을 갖는다. 해당 이미지를 여러번 빌드하다보면 다음번 빌드 이미지에 이름을 빼앗기는 경우가 많다.    
이런 이름 없는 이미지를 `dangling image`라고 한다.   

```
# dangling image 삭제
$ docker image prune

# 컨테이너에서 사용하고 있지 않은 이미지들을 삭제
$ docker image prune -a

# 이미지를 사용하고 있는 컨테이너 찾기
$ docker ps -a --filter ancestor=nginx:alpine

# 이미지를 사용하고 있는 컨테이너를 찾아 종료하고 이미지 삭제
$ docker rm -f $(docker ps -aq --filter ancestor=nginx:alpine)

# 모든 이미지 제거
$ docker rm -f $(docker ps -aq)

# 중지된 지 7일 이상 지난 이미지 삭제
$ docker image prune --filter until=7d

# 라벨이 존재하는 이미지 삭제
$ docker image prune --filter label=env

# 라벨이 존재하지 않는 이미지 삭제
$ docker image prune --filter label!=env

# 라벨의 값을 확인하여 이미지 삭제
$ docker image prune --filter label=env=development

# 라벨의 값을 확인하여 다르다면 이미지 삭제
$ docker image prune --filter label!=env=production
```

### docker vomule prune

컨테이너와 연결되어있지 않은 모든 볼륨 오브젝트 삭제

### docker network prune

컨테이너와 연결되어있지 않은 모든 네트워크 오브젝트 삭제

### docker system prune

컨테이너, 이미지, 볼륨, 네트워크 각각의 Docker 오브젝트 삭제
