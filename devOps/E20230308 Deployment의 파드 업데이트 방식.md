## Deployment의 파드 업데이트 방식

- Rolling update : 하나씩 내리고 새로운 이미지로 업데이트 / 상이한 버전이 공존하는 문제
- Recreate : 전부 내리고, 전부 재생성
- Blue/Green : 똑같은 자원만큼 새로운 서비스를 띄우고 전부 돌림 / 버전 공존 문제 X, 롤백 O
- Canary : 조금씩 자원을 업데이트하고 트래픽을 분산 
