### 전체 구조

![image](https://github.com/ecsimsw/pic-up/assets/46060746/9fafa2c1-ea3d-4853-b2e8-87045b6dae09)

### spec
- vCpu : 2
- vMem : 4GB
- OS : ubuntu 22.04

### version
- kubernetes v1.28
- CRI-O v1.28
- Calico v3.25

### VM, Vagrant 
- 쿠버네티스를 설치할 때 Swap 을 비활성화 해줘야한다. 
- 쿠버네티스가 리소스를 관리할 때 노드에서 발생하는 Swap 은 고려하지 않기에 비활성화를 권장한다고 한다.
- 배포 서버를 단일 노드로 k8s 를 설치하려고 했으나 배포 서버 자체는 Swap 를 비활성화하고 싶지 않았다.
- 또 k8s 설치나 설정에 문제가 발생하면 편하게 리셋하고 싶어서 VM을 만들어 환경을 분리했다.
- VM 설정을 GUI 없이 코드와 커맨드로만 관리하고 싶어서 vagrant 를 사용한다.

### Vagrant bridge
- 그림 아래에 회색 Bridge는 Vagrant public network 를 가능하게 한다.
- 동일한 Router 아래서 할당된, 즉 서브넷 192.168.0.0/24 안에 포함되는 모든 기기에서 VM에 접근할 수 있도록 한다.
- 덕분에 개발에 사용하는 외부 PC에서 쉽게 접근하거나, 같은 서브넷에 포함된 pc를 worker node로 등록이 가능하다.
- [blog.jeffli.me/vagrant-networking](https://blog.jeffli.me/blog/2017/04/22/vagrant-networking-explained/)

### Ingress와 Node port
- Service 마다 외부 IP 를 갖을 수 있는 상황이 아니니, Port 를 지정하고 Router 에서 포트 포워딩으로 외부 접근을 열어둘 생각이다.
- 문제는 Service 마다 Port를 매번 뚫기엔 자원도 아깝고, 관리도 비효율적이다. 
- 전면에 Ingress 를 두고 Ingress 하나만 Port 를 뚫어둔 후, 요청 Path 로 Service 구분하여 전달할 생각이다.
- 기존 서버스별 포트 설정을 변경하지 않으면서도 외부에 노출되는 Port 를 아끼고, 관리 포인트를 줄인다.

### Volume, NFS
- Home server 의 특정 경로에 데이터(ex, Logs, CDN private_key)을 모을 생각이다. 
- 특정 경로를 NFS로 등록하고, Dynamic Provisioning 으로 다른 PV 생성없이 Pod에서 필요한 볼륨을 얻고 사용할 수 있도록 한다.
- [kubernetes-sigs/nfs-subdir-external-provisioner](https://github.com/kubernetes-sigs/nfs-subdir-external-provisioner) 

### Ref
- [ecsimsw/readme](https://github.com/ecsimsw/pic-up/tree/main/infra-kubernetes)
- [kubernetes-docs/kubectl](https://kubernetes.io/ko/docs/tasks/tools/install-kubectl-linux/)
- [shape.host/cri-o](https://shape.host/resources/cri-o-container-runtime-installation-guide-ubuntu-22-04)
- [docs.tigera.io/Calico on Kubernetes](https://docs.tigera.io/calico/latest/getting-started/kubernetes/quickstart)

### Deployment strategy
1. Rolling update : 점진적 확장과 제거
maxUnavailable, maxSurge 으로 제거 비율 (개수), 확장 비율 (개수) 지정
여러 버전 공존 문제
2. Recreate : 기존 파드 전부 제거 후 재생성, 
다운 타임 문제
3. Rollback 처리 
Deployment 가 이전 ReplicaSet 을 제거하지 않고 롤백에 사용
4. Blue-Green : ReplicaSet 개수만큼 파드 확장 후 한번에 방향 전환
전면 LB 라우팅 규칙을 바꾸거나, Deployment 를 따로 생성 후 Service 의 selector 를 바꿔 전환으로 처리
5. Canary : 일부 비율의 사용자만 Update 된 pod로
전면에서 사용자를 구분할 수 있는 방법 필요, 예를 들면 IP 라던가, Header, 쿠키

### Pod scheduling strategy
- NodeName
- NodeSelector
- NodeAffinity
- PodAffinity
- PodAntiAffinity
- Taints, Tolerations

### Probes
1. startupProbe : 성공할 때까지 다른 Probe 를 실행하지 않는다. 실패하면 컨테이너를 죽이고 재시작 정책에 따른다.
2. readinessProbe : 요청을 처리할 수 있는지 헬스 체크. 비정상 응답 또는 Timeout 시 Service에서 제거
3. livenessProbe : 컨테이너가 동작하는지 헬스 체크. 실패하면 컨테이너를 죽이고 재시작 정책에 따른다.

### Service 
- Pod는 내부 IP를 갖고, Pod의 수나 Image, 정책은 ReplicaSet과 Deployment에 의해 관리되지만 매번 새로 IP 가 할당된다.
- Service는 이런 Pod가 고정된 단일 진입점을 갖을 수 있도록 한다. 
- Service는 고정된 IP를 갖도록 하고, 지정된 pod 들에게 요청을 분산하는 것으로 외부와 고정된 통신이 가능하다.
- 이때 Service는 Pod를 IP:PORT가 아닌 label Selector 를 이용해서 찾는다. (파드 IP가 매번 바뀌는게 문제였으니)

### Service 타입 
- ClusterIP : 내부 IP
- NodePort : 클러스터의 모든 노드에 특정 포트를 열어 Forwarding
- LoadBalancer : 클라우드 제공자의 LB로 IP를 할당 받아서 사용, AWS ALB를 Service LB로 연결해서 ip 할당받아 사용해봄 
- On premise 에선 metal LB로 LB를 흉내냈었는데..

### Volume
- emptyDir : 파드내 컨테이너간 데이터 공유, 워커 노드의 Disk 나 메모리, 일시적
- hostPath : 노드에 데이터를 영속, 스케줄링으로 노드 고정하여 동일한 파일 사용
- 논리 스토리지, 인프라 분리 : 관리자가 PV 생성, 개발자가 파드 PVC로 사용할 PV 할당
- persistentVolumeReclaimPolicy : PV 사용 종료시 데이터를 어떻게 할 것인가
- Retain : PVC가 삭제되어도 데이터 보존, 대신 다른 PVC가 사용X, 수동으로 PV를 제거 후 재생성해야 연결 가능
- Delete : PVC가 삭제되면 PV와 데이터를 모두 제거
- Dynamic Provisioning : CSI를 지원하는 Storage provider를 사용해서 PVC로 볼륨만 요청하면 PV가 자동으로 생성되어 Bound 될 수 있도록 (NFS, EBS, EFS..)

### Resource - Request, Limit
- Container의 cpu, mem request 합을 수용 가능한 노드에 프로비저닝된다. 
- 기본적으로는 지정 노드가 없다면 수용 가능한 노드들에서 중 RR으로 스케줄링된다.
- 컨테이너의 cpu 사용량이 limit 에 도달하게 되면 스로틀링이 일어난다. 
- 컨테이너의 메모리 사용량이 limit 에 도달하면 컨테이너는 종료된다. (실행 가능 상태가 되면 Kubelet에 의해 재실행)
- Request 를 적게 잡으면 프로비저닝이 쉬워 자원을 효율적으로 스케줄링 문제를 덜하고 사용할 수 있겠지만, 런타임 중에 사용량이 늘면 전체 OverCommit 이 발생할 확률이 커진다.
- Limit 를 크게 잡으면 런타임 중에 컨테이너가 사용할 수 있는 리소스 범위는 커져 전체 노드 OverCommit 이 발생할 확률이 커지지만, Limit을 작게 잡으면 컨테이너가 사용할 수 있는 자원이 작아 Evict 과 재실행이 반복된다.

### OOM killer 
- CPU는 한계를 넘어버리면 압축이 가능하지만 메모리는 그렇지 못하다. 
- limit은 파드 스케줄링과 관련이 없어, 런타임 도중 노드에 메모리 부족이 발생할 수 있다. (기본, 가용 메모리 100Mi 이하일 때) 
- 이때 OOM Killer 가 실행되어 컨테이너/파드를 정리한다. 이때 퇴거 우선 순위는 QoS 클래스에 따라 다르다.

[kubernetes-docs/memory-resource](https://kubernetes.io/ko/docs/tasks/configure-pod-container/assign-memory-resource/)
