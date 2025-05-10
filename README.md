# Daily notes    
### Opened issues
- 좋아요 수로 정렬 + 페이징 구현 전략
- 토스 뱅크 - 캐시를 적용하기 까지의 험난한 길 (TPS 1만 안정적으로 서비스하기)
- 카카오 페이 - JPA Transactional 잘 알고 쓰고 계신가요?
- Amazon ECS Service Connect를 활용하여 손쉽게 마이크로서비스 운영하기
- Lambda와 WAF를 이용한 Rate-Based Blacklisting 기능 구현
- 리눅스 서버의 TCP 네트워크 성능을 결정짓는 커널 파라미터 이야기 : NHN Cloud

### Closed issue
- KafkaTemplate 동작 구조
- 레디스 동시성 문제, 두번의 갱실 분실 문제 소개하기
- 온프라미스 Ubuntu, Thingsboard config 정리
- 회사 서버 OOM 문제 모니터링, 원인 분석과 대기열 구조 개선
- Logging 시스템 개선, Grafana labs LGTM 스택 도입기
- EDA 구조 고민, SNS + SQS 조합 분석
- Eclipse memory analyzer, 특정 자바 객체가 얼마나 많은 메모리를 사용하고 있는지 확인
- ECS Terraform IaC
- AWS에서 어떤 컨테이너 서비스를 이용해야 하나요?
- Spring pulsar listener로 Pulsar java client 리팩토링
- Tuya message service, About pulsar
- 당근페이 - 확장성 있는 TCP 통신 시스템 구축하기
- 올리브영 셔터 이미지 업로드 성능 개선기
- 경쟁력 있는 개발자를 위한 '클라우드 디자인 패턴' - 엘키
- CJ 올리브영의 서버리스 랭킹 시스템 구축기
- YOUNGJAE.KIM - 우리 회사의 기업문화에 대해
- Spring N차 캐시 구현
- AWS를 활용한 확장성 높은 모바일 트레이딩 시스템 (MTS) 구축하기
- ALB, Lambda 전면에 CloudFront 를 두는 이유
- HTTP2.0 설정
- Mysql MVCC, GAP LOCK 실습
- Mysql db replication / spring multi db source config
- RabbitMQ - Dead Letter Exchange
- Make EKS endpoint private with VPN
- DB bulk insert시 확인하면 좋을 것들
- How to Choose the Number of Topics/Partitions in a Kafka
- How can prepared statements protect from sql injection
- Brady - 사용중인 DB 커넥션 수 확인하기
- Cursor based pagination compared to offset based
- The best way to use the Spring Data JPA Specification
### Index
    .
    ├── aws
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
    │   ├── E20230602 Cloudwatch slack alert with SNS, Lambda, Chatbot.md
    │   ├── E20240302 route53 으로 domain 장애 조치.md
    │   ├── E20240410 CloudFront jwt decoder function.md
    │   └── E20240410 CloudFront signed url.md
    ├── backend
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
    │   ├── E20240104 Nginx LB.md
    │   ├── E20240104 install nginx.md
    │   ├── E20240202 Animekr 재배포하면서...md
    │   ├── E20240222 SOP, CORS.md
    │   ├── E20240222 XSS, CSRF.md
    │   ├── E20240302 AUTO_INCREAMENT로 batch insert 가 안됐던 이유.md
    │   ├── E20240306 해시맵의 동시성 문제.md
    │   ├── E20240307 AtomicIneteger CAS 어떻게 동작하는지.md
    │   ├── E20240307 내 서비스를 예시로 Redis, RabbitMQ, Kafka.md
    │   ├── E20240307 레디스 pub,sub.md
    │   ├── E20240321 named 락에 Transactional new.md
    │   ├── E20240328 비동기, Non blocking.md
    │   ├── E20240403 Thumbnailator 최대 사이즈 필터링.md
    │   ├── E20240407 HikariCP의 Connection check.md
    │   ├── E20240418 java foreach linkedList.md
    │   ├── E20240425 spring boot rabbitmq settings.md
    │   ├── E20240428 영속성컨텍스트와 OSIV.md
    │   ├── E20240504 hashicorp Vault.md
    │   ├── E20240517 Vault injector for k8s.md
    │   ├── E20240528 대표적인 IPC.md
    │   ├── E20240528 서버 분리와 DB 분리.md
    │   └── E20240530 Cloudfront Signed url 생성 - AWS  SDK Java V2.md
    ├── cs
    │   ├── db
    │   │   ├── E20221007 NOSQL vs RDBMS.md
    │   │   ├── E20221008 ACID.md
    │   │   ├── E20221009 Why NoSql is better at scaling out.md
    │   │   ├── E20221010 DBMS의 데이터 버퍼.md
    │   │   ├── E20221010 JPA Transactional readOnly.md
    │   │   ├── E20221016 Redis 아는 척하기.md
    │   │   ├── E20221016 캐시 전략.md
    │   │   ├── E20230523 Sharding, does it still need to search all the sharding database servers?.md
    │   │   ├── E20230722 DB file size.md
    │   │   ├── E20230723 DB 인덱싱.md
    │   │   ├── E20230725 DB 조회 실행 순서.md
    │   │   ├── E20231116 create mysql user for specific ip.md
    │   │   ├── E20231230 mongosh 명령어 모음.md
    │   │   ├── E20240220 table based lock 을 row lock 으로.md
    │   │   ├── E20240314 Mysql MVCC.md
    │   │   ├── E20240314 gap lock.md
    │   │   ├── E20240321 실행계획으로 보는 것.md
    │   │   ├── E20240321 클러스터드 인덱스, 넌클러스터드 인덱스.md
    │   │   ├── E20240327 머릿속 인덱스 정리하기.md
    │   │   ├── E20240328 머릿속 트랜잭션 정리.md
    │   │   ├── E20240531 DB insert.md
    │   │   └── E20240624 Mysql backup.md
    │   ├── network
    │   │   ├── 20240128 HTTP 버전.md
    │   │   ├── E20220918 TCP Handshake.md
    │   │   ├── E20221002 CIDR.md
    │   │   ├── E20221003 TCP-IP 4계층.md
    │   │   ├── E20221010 DNS- A record와 CNAME의 차이.md
    │   │   ├── E20221112 NAS란.md
    │   │   ├── E20221125 VPC를 구성하는 가장 기본적인 방법.md
    │   │   ├── E20230531 Https 원리.md
    │   │   ├── E20240309 누군가 나에게 Rest API 가 뭔지 물어본다면.md
    │   │   └── E20240414 Content-cache.md
    │   └── os
    │       ├── E20220915 블록킹과 논블록킹, 동기와 비동기.md
    │       ├── E20220916 컴퓨터가 부팅되는 과정.md
    │       ├── E20220919 Containers vs Virtual machines.md
    │       ├── E20221014 FD, Redirection, Pipe.md
    │       ├── E20221014 nohup과 &.md
    │       ├── E20230117 Expand Proxmox Disk.md
    │       ├── E20230225 Authorized_keys와 known_hosts.md
    │       ├── E20230303 Interrupt.md
    │       ├── E20230510 Linux kill.md
    │       ├── E20230605 htop으로 모니터링.md
    │       ├── E20230605 linux cpu info.md
    │       ├── E20230630 arp scan.md
    │       ├── E20230705 What is bin sh -c.md
    │       ├── E20231114 docker-on-ubuntu.md
    │       └── E20241222 IO바운드 작업과 CPU 바운드 작업.md
    ├── ops
    │   ├── deployment
    │   │   ├── E20220915 Jenkins shared library.md
    │   │   ├── E20220923 Jenkins generic webhook trigger plugin.md
    │   │   ├── E20221012 젠킨스 ContainerTemplate 함수화하기.md
    │   │   ├── E20221026 Apply SonarScanner on Jenkins.md
    │   │   ├── E20230413 jenkins auto re-run script.md
    │   │   └── E20230414 Jenkins agent health checker.md
    │   ├── kubernetes
    │   │   ├── E20221102 k8s의 볼륨.md
    │   │   ├── E20221103 k8s의 PV, PVC.md
    │   │   ├── E20221201 How to create user, role on K8S.md
    │   │   ├── E20221206 Master node, worker node .md
    │   │   ├── E20221206 Pod.md
    │   │   ├── E20230111 On premise K8S cluster 구성하기.md
    │   │   ├── E20230111 Why you can’t ping a Kubernetes Service.md
    │   │   ├── E20230114 How to configure cluster on proxmox.md
    │   │   ├── E20230115 DNS, K8S Core DNS.md
    │   │   ├── E20230308 Deployment의 파드 업데이트 방식.md
    │   │   ├── E20230308 매번 헷갈리는 Replicaset, Daemonset, Statefulset, Deployment.md
    │   │   ├── E20230616 k8s 세가지 probe.md
    │   │   ├── E20230616 statefulset.md
    │   │   ├── E20230713 PodDisruptionBudget.md
    │   │   ├── E20230714 Core dns name policy.md
    │   │   ├── E20230719 Service port.md
    │   │   ├── E20231115 Kubernetes volume 과 NFS 설정.md
    │   │   ├── E20240517 Install k8s v1.28.md
    │   │   └── E20240530 picup k8s cheat sheet.md
    │   ├── E20220915 Git submodule.md
    │   ├── E20221019 How to migrate multiple repo to mono repo.md
    │   ├── E20221102 Git LFS.md
    │   ├── E20230511 github-actions-self-hosted 사용 전략.md
    │   ├── E20230714 Github action gradle cache.md
    │   ├── E20230720 submodule fetch erorr.md
    │   ├── E20230806 git filter all repo.md
    │   ├── E20240103 GHCR.md
    │   ├── E20240104 pr file change validator.md
    │   ├── E20240105 git chmod no track.md
    │   ├── E20241101 Ubuntu user init.md
    │   ├── E20241101 proxy 서버 ngrep.md
    │   ├── E20241211 docker group.md
    │   ├── E20250323 Git migration.md
    │   └── E20250418 ubuntu init script.md
    ├── tips
    │   ├── E20221007 Docker prune.md
    │   ├── E20221012 IntelliJ - Column selection.md
    │   ├── E20221130 Minikube 설정기.md
    │   ├── E20230105 Autosuggestions.md
    │   ├── E20230111 Git config commit author.md
    │   ├── E20230114 Autojump.md
    │   ├── E20230119 Itsycal - calandar.md
    │   ├── E20230127 Clipy.md
    │   ├── E20230215 Git author tip.md
    │   ├── E20230220 K9S.md
    │   ├── E20230313 .DS_Store.md
    │   ├── E20230318 Turn off app store notification.md
    │   ├── E20230323 하위 폴더들이 차지 하고 있는 디스크 용량.md
    │   ├── E20230329 Find using port by PID.md
    │   ├── E20230330 Check web page has changed.md
    │   ├── E20230414 My terminal commands.md
    │   ├── E20230417 Vertual Network Computing.md
    │   ├── E20230418 drwxr-xr-x.md
    │   ├── E20230513 My ubuntu dock conf.md
    │   ├── E20230605 ecsimsw-server zsh config.md
    │   ├── E20230605 my server banner.md
    │   ├── E20230609 MAC - flush dns cache.md
    │   ├── E20230618 ghcr.md
    │   ├── E20230618 vnc server on ubuntu.md
    │   ├── E20230624 xcrun: error: invalid active developer.md
    │   ├── E20230630 Semantic Versioning regex.md
    │   ├── E20230630 공유기가 두개인 경우.md
    │   ├── E20230702 checkip.amazonaws.com.md
    │   ├── E20230705 switch java version.md
    │   ├── E202311014 ubuntu new user 세팅.md
    │   ├── E20240103 ubuntu 22 docker install.md
    │   ├── E20240105 TLS nginx.md
    │   ├── E20240108 docker prune all.md
    │   ├── E20240211 Intellij refactor options.md
    │   ├── E20240224 disable sonoma input lang icon.md
    │   ├── E20240302 ubuntu delete nginx.md
    │   ├── E20240319 reset Grafana user .md
    │   ├── E20240516 Pem 키를 환경 변수로 넣어야할 때.md
    │   ├── E20240812 Check list.md
    │   ├── E20240812 Mac background items added.md
    │   ├── E20240814 Terminal username and model.md
    │   └── E20240924 자바 프로세스, 포트 목록 명령어.md
    ├── closed_issues.txt
    ├── open_issues.txt
    └── updated_readme.md
