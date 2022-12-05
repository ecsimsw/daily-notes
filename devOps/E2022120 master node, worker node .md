# Master, Worker node

![image](https://user-images.githubusercontent.com/46060746/205706345-08648743-8b37-44b6-9515-ace70134249d.png)

Kubelet of master node : Container를 관리한다. 이 Container 안에 Scheduler, Proxy, api-server, Container manager가 있다.   

Kubelet of worker node : Master node의  API server를 바라보고, Pod의 생성/관리/삭제를 담당한다.

etcd : key-value 저장소이다.

Kube scheduler : Pod가 생성될 때, Pod의 자원 요청 정보에 따라 Cluster 내 알맞은 자원(노드)를 찾고 Pod를 그 곳에 띄워주는 역할을 한다.

Kube controller manager : Pod를 관리하는 Controller를 관리하는 역할을 한다. Controller 는 Scheduler를 참고하여 Pod를 실행하고, Pod에 문제가 생기면 이를 감지하여 대응한다. Service를 Pod에 연결하여 엔드포인트를 만들기도 한다.

Kube-proxy : 클러스터 내부에 가상 네트워크를 설정/관리한다.

Container runtime : 실제로 컨테이너를 실행시킨다. Docker, runc, rkt 등이 있다.

### REF

https://arisu1000.tistory.com/27827   
https://www.redhat.com/ko/topics/containers/kubernetes-architecture   
https://velog.io/@squarebird/Kubernetes-Kubernetes-구조
