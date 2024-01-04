# My daily note    
![main](./.images/main.jpg)    

    .
    ├── AWS
    │   ├── E20220916 AWS ECR.md
    │   ├── E20220921 VPC 구성하기와 지원되지 않는 VPC 구성.md
    │   ├── E20221112 Route53과 ALB로 급한 불 끄기.md
    │   ├── E20221123 Create private domain with route53.md
    │   ├── E20221125 ACL과 SG의 차이.md
    │   ├── E20230104 Cloudfront S3.md
    │   ├── E20230313 AWS 자원 생성 시에 태깅을 강제하자.md
    │   ├── E20230413 Access EKS.md
    │   ├── E20230418 ECR KMS encryption.md
    │   ├── E20230514 Live EKS cluster 구축하며 생긴 일.md
    │   ├── E20230531 EKS Cloudwatch agent.md
    │   └── E20230602 Cloudwatch slack alert with SNS, Lambda, Chatbot.md
    ├── CICD
    │   ├── E20220915 Jenkins shared library.md
    │   ├── E20220923 Jenkins generic webhook trigger plugin.md
    │   ├── E20221012 젠킨스 ContainerTemplate 함수화하기.md
    │   ├── E20221026 Apply SonarScanner on Jenkins.md
    │   ├── E20230413 jenkins auto re-run script.md
    │   ├── E20230414 Jenkins agent health checker.md
    │   ├── E20230511 github-actions-self-hosted 사용 전략.md
    │   ├── E20230714 Github action gradle cache.md
    │   └── E20240104 pr file change validator.md
    ├── DB
    │   ├── E20221007 NOSQL vs RDBMS.md
    │   ├── E20221008 ACID.md
    │   ├── E20221009 Why NoSql is better at scaling out.md
    │   ├── E20221010 DBMS의 데이터 버퍼.md
    │   ├── E20221010 JPA Transactional readOnly.md
    │   ├── E20221016 Redis 아는 척하기.md
    │   ├── E20221016 캐시 전략.md
    │   ├── E20230523 Sharding, does it still need to search all the sharding database servers?.md
    │   ├── E20230722 DB file size.md
    │   ├── E20230723 DB 인덱싱.md
    │   ├── E20230725 DB 조회 실행 순서.md
    │   ├── E20231116 create mysql user for specific ip.md
    │   └── E20231230 mongosh 명령어 모음.md
    ├── DEV
    │   ├── E20220921 인증과 인가.md
    │   ├── E20221014 Spring MVC DTO에 Setter가 필요한 경우.md
    │   ├── E20221024 Elasticsearch 아는 척하기.md
    │   ├── E20221025 Fluentd에서 Filebeats + Logstash로 이전.md
    │   ├── E20221025 공개키와 개인키.md
    │   ├── E20221031 Kotlin의 시퀀스.md
    │   ├── E20221112 OTP.md
    │   ├── E20221127 JAR와 WAR.md
    │   ├── E20230418 내가 써본 부하 테스트 툴.md
    │   ├── E20230514 나의 인증 토큰 관리 방법.md
    │   ├── E20230518 SHA256.md
    │   ├── E20230604 Spring TransactionalEventListener.md
    │   ├── E20230604 spring & docker kafka config.md
    │   ├── E20230604 spring & docker mongo config.md
    │   ├── E20230607 가장 간단한 웹서버.md
    │   ├── E20230620 Kafka partition key, group id.md
    │   ├── E20230717 Hibernate log properties.md
    │   ├── E20230717 Persistence context with 용관.md
    │   ├── E20230720 Springboot - change loglevel at runtime.md
    │   ├── E20230725 Nginx skeleton.md
    │   ├── E20230805 java String length.md
    │   ├── E20230809 서버 개발 시큐어 가이드.md
    │   ├── E20230812 Spring remote ip.md
    │   ├── E20231120 Redirecting post to post.md
    │   └── E20240104 install nginx.md
    ├── ENV
    │   ├── E20221130 Minikube 설정기.md
    │   ├── E20230105 Autosuggestions.md
    │   ├── E20230111 Git config commit author.md
    │   ├── E20230114 Autojump.md
    │   ├── E20230119 Itsycal - calandar.md
    │   ├── E20230127 Clipy.md
    │   ├── E20230220 K9S.md
    │   ├── E20230313 .DS_Store.md
    │   ├── E20230318 Turn off app store notification.md
    │   ├── E20230513 My ubuntu dock conf.md
    │   └── E20240103 ubuntu 22 docker install.md
    ├── Git
    │   ├── E20220915 Git submodule.md
    │   ├── E20221019 How to migrate multiple repo to mono repo.md
    │   ├── E20221102 Git LFS.md
    │   ├── E20230720 submodule fetch erorr.md
    │   ├── E20230806 git filter all repo.md
    │   └── E20240103 GHCR.md
    ├── Kubernetes
    │   ├── E20221102 k8s의 볼륨.md
    │   ├── E20221103 k8s의 PV, PVC.md
    │   ├── E20221201 How to create user, role on K8S.md
    │   ├── E20221206 Master node, worker node .md
    │   ├── E20221206 Pod.md
    │   ├── E20230111 On premise K8S cluster 구성하기.md
    │   ├── E20230111 Why you can’t ping a Kubernetes Service.md
    │   ├── E20230114 How to configure cluster on proxmox.md
    │   ├── E20230115 DNS, K8S Core DNS.md
    │   ├── E20230308 Deployment의 파드 업데이트 방식.md
    │   ├── E20230308 매번 헷갈리는 Replicaset, Daemonset, Statefulset, Deployment.md
    │   ├── E20230616 k8s 세가지 probe.md
    │   ├── E20230616 statefulset.md
    │   ├── E20230713 PodDisruptionBudget.md
    │   ├── E20230714 Core dns name policy.md
    │   ├── E20230719 Service port.md
    │   └── E20231115 Kubernetes volume 과 NFS 설정.md
    ├── Network
    │   ├── E20220918 TCP Handshake.md
    │   ├── E20221002 CIDR.md
    │   ├── E20221003 TCP-IP 4계층.md
    │   ├── E20221010 DNS- A record와 CNAME의 차이.md
    │   ├── E20221112 NAS란.md
    │   ├── E20221125 VPC를 구성하는 가장 기본적인 방법.md
    │   └── E20230531 Https 원리.md
    ├── OS
    │   ├── E20220915 블록킹과 논블록킹, 동기와 비동기.md
    │   ├── E20220916 컴퓨터가 부팅되는 과정.md
    │   ├── E20220919 Containers vs Virtual machines.md
    │   ├── E20221014 FD, Redirection, Pipe.md
    │   ├── E20221014 nohup과 &.md
    │   ├── E20230117 Expand Proxmox Disk.md
    │   ├── E20230225 Authorized_keys와 known_hosts.md
    │   ├── E20230303 Interrupt.md
    │   ├── E20230323 하위 폴더들이 차지 하고 있는 디스크 용량.md
    │   ├── E20230329 Find using port by PID.md
    │   ├── E20230418 drwxr-xr-x.md
    │   ├── E20230510 Linux kill.md
    │   ├── E20230605 htop으로 모니터링.md
    │   ├── E20230605 linux cpu info.md
    │   ├── E20230630 arp scan.md
    │   ├── E20230705 What is bin sh -c.md
    │   └── E20231114 docker-on-ubuntu.md
    └── Tip
        ├── E20221007 Docker prune.md
        ├── E20221012 IntelliJ - Column selection.md
        ├── E20230215 Git author tip.md
        ├── E20230330 Check web page has changed.md
        ├── E20230414 My terminal commands.md
        ├── E20230417 Vertual Network Computing.md
        ├── E20230605 ecsimsw-server zsh config.md
        ├── E20230605 my server banner.md
        ├── E20230609 MAC - flush dns cache.md
        ├── E20230618 ghcr.md
        ├── E20230618 vnc server on ubuntu.md
        ├── E20230624 xcrun: error: invalid active developer.md
        ├── E20230630 Semantic Versioning regex.md
        ├── E20230630 공유기가 두개인 경우.md
        ├── E20230702 checkip.amazonaws.com.md
        ├── E20230705 switch java version.md
        └── E202311014 ubuntu new user 세팅.md
