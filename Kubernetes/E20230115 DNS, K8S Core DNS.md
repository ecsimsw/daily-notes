# DNS, K8S Core DNS

## /etc/hosts

linux에서 도메인의 IP를 찾을 때 가장 먼저 확인하는 파일

```
jinhwan@jinhwan-ubuntu:~$ cat /etc/hosts
127.0.0.1	localhost
127.0.1.1	jinhwan-ubuntu

# The following lines are desirable for IPv6 capable hosts
::1     ip6-localhost ip6-loopback
fe00::0 ip6-localnet
ff00::0 ip6-mcastprefix
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters
```

## /etc/resolv.conf

 DNS resolver, DNS 서버 목록

## Kubernetes의 DNS

coredns라는 pod이 kube-system으로 동작한다. 이 pod는 클러스터를 지속적으로 모니터하고 새로운 service or pod이 생성될 경우 이를 도메인 서버에 업데이트 한다.    

pod이 생성될 때 해당 pod의 /etc/resolv.conf에 coreDNS 서버의 IP를 추가하여 `my-service.default.svc.cluster.local`같은 클러스 내부의 DNS를 사용할 수 있도록 한다. 
