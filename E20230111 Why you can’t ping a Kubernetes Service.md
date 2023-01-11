# Why you can’t ping a Kubernetes Service.md

https://nigelpoulton.com/why-you-cant-ping-a-kubernetes-service/

### ICMP 
ICMP(Internet Control Message Protocol)은 TCP/IP의 프로토콜. Ping은 이 프로토콜을 사용하여 네트워크 장치간 연결을 확인한다.

### K8S 

```
A Kubernetes Service is a stable networking endpoint that sits in front of a set of application Pods. Instead of accessing Pods directly you access them through the Service. The Service exposes a DNS name, virtual IP, and network port that you can use to connect to the Pods behind it.
```

K8S의 Pod은 생성될 때마다 매버 새로운 IP를 갖기 때문에 이 IP로 네트워크를 유지하기 어렵다. 서비스는 파드를 묶어 네트워크의 진입점을 만는다. 개별적인 POD의 IP를 진입점으로 하느 것이 아닌, Service라는 하나의 진입점을 갖도록 하고 이것이 여러 파드를 묶는 것이다. 

```
The short reason is that a Kubernetes Service only activates when connections arrive on the correct port. Unfortunately ping doesn’t use ports 🙁
```

K8S의 service는 IP와 Port를 필요로 하고, ICMP는 포트를 사용하지 않는다. 그래서 K8S는 ping이 불가.
