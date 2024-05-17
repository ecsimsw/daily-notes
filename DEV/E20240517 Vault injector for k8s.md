### Secret 관리
- DB 비밀번호, AWS 키, 토큰과 AES 암호화 시크릿 키 등 공개되어선 안되는 비밀 값들을 관리할 수 있는 방법을 고민한다.
- 지금은 환경 변수를 세팅하는 쉘 스크립트와 k8s secret 값을 만들어두고 git ignore으로 숨기고 있는데 값 관리가 어렵고 실행이 번거롭다.
- 비밀 키 값, 환경 변수를 관리하는 저장소를 구성하고, 비밀 키 값이 포함된 쉘 스크립트나 파일들을 제거한다.

### Private repository, 
- public 으로 올리지 않는 파일만 private repository 에 따로 관리하여 숨긴다.
- 암호화된 파일을 아예 분리된 경로에 위치시켜야해서 파일이 분산되고 디렉토리 관리가 편하지 않았다.
- 다른 도구가 불필요해 설정이 쉽다.

### Submodule + Private
- 키 값, 환경 변수 값 저장의 버전 문제를 풀이할 때 사용한 경험이 있다. 
- 예를 들어 A,B 프로젝트에서 공통 저장소로 Private repository 를 사용하는 도중, A가 업데이트 되면서 키 값이 서로 달라질 수 있다.
- Submodule은 특정 commit 을 끌어와 사용하기에 위 상황에서 A와 B는 공통된 저장소를 사용하나 서로에게 유효한 값을 가진 Commit 을 바라볼 수 있게된다.
- 개인적으로는 사용이 불편했다. 팀에서 모두 submodule 을 신경써야하고 (사용 방법, 사용 여부, Commit 버전 등), 충돌이나 꼬이면 푸는게 귀찮았다.
- 다른 도구 없이 Git 으로 해결할 수 있어 편하면서도 Private repo 로도 풀 수 있으면 굳이 싶다.

### Vault
- 키 관리 저장소를 따로 둔다.
- 인증부와 키 자체를 분리하여 사용자와 권한 범위를 관리하기 좋다. (퇴사자 문제)
- Dynamic secret이나 TTL 으로 권한의 유효 기간을 두거나 권한 자체를 임시로 생성 - 제거할 수 있어 안전하다.
- AWS Dynamic secret 으로 CI/CD에 필요한 AWS 권한만을 갖는, 작업 기간 동안만 유효한 임시 키를 사용한 경험이 있다.
- 다양한 인증 방법과 저장 공간(Storage engine), 관리 가능한 키를 제공하고, 참고할 수 있는 자료가 많다는 것도 큰 강점이었다.

### Vault injector (Side car pattern) 
1. Pod 가 생성될 때 Init container에서 Service account(token)로 Vault k8s auth 에 로그인을 요청한다.
2. Vault agent container는 발급 받은 로그인 토큰과 함께 필요한 Secret 값을 요청한다.
3. Vault에선 로그인 토큰의 유효함, 사용자의 권한을 확인한다.
4. 요청한 Secret 값을 전달하고, Vault agent 는 이를 임시 저장 공간에 저장한다.
5. Application container에서 저장된 Secret 값을 참조하여 환경 변수를 구성한다.

![image](https://github.com/ecsimsw/pic-up/assets/46060746/76357f42-e001-4800-8953-9ea2e4469470)

### Configuration
1. Helm values.yaml 준비
- KUBERNETES_IP : 쿠버네티스 host ip
- PORT : 쿠버네티스 port
``` yaml
global:
  externalVaultAddr: "${KUBERNETES_IP:PORT}
injector:
  enabled: "true"
  port: ${PORT}
```
2. Vault-agent-injector 설치
```
helm repo add hashicorp https://helm.releases.hashicorp.com
helm install vault -f values.yaml hashicorp/vault
```
3. Vault kubernetes 인증에 사용할 Secret 생성 
- SECRET_NAME : 생성할 secret 이름
``` yaml
apiVersion: v1
kind: Secret
metadata:
  name: $SECRET_NAME
  annotations:
    kubernetes.io/service-account.name: vault
type: kubernetes.io/service-account-token
```
4. Vault kubernetes 인증 생성
- TOKEN_REVIEWER_JWT : 앞서 생성한 secret의 token 값
```
TOKEN_REVIEWER_JWT=$(kubectl get secret $SECRET_NAME --output='go-template={{ .data.token }}' | base64 --decode)
KUBE_CA_CERT=$(kubectl config view --raw --minify --flatten --output 'jsonpath={.clusters[].cluster.certificate-authority-data}' | base64 --decode)
KUBE_HOST=$(kubectl config view --raw --minify --flatten -o jsonpath='{.clusters[].cluster.server}')
```
```
vault auth enable kubernetes

vault write auth/kubernetes/config  \
   token_reviewer_jwt="$TOKEN_REVIEW_JWT} \
   kubernetes_ca_cert="$KUBE_CA_CERT" \
   kubernetes_host="$KUBE_HOST"
```
5. Vault Secret 생성, 값 추가
```
ex,
vault secrets enable -path=picup kv
vault kv put picup/common version=1 created=20240516
```

6. Vault Policy 정의 (Secret 사용 범위 지정)
```
ex,
path "picup/*" {
  capabilities = ["read", "list"]
}
```
```
vault policy write {POLICY_NAME} {POLICY_FILE_PATH}
```
7. Vault Auth role 정의
- ROLE_NAME : 생성할 Role 이름
- SERVICE_ACCOUNT_NAME_OF_POD : Vault agent 가 생성될 Pod 의 Service account 이름
- POD_NAME_SPACE : Vault agent 가 생성될 Pod 의 namespace
- POLICY_NAME : 미리 정의한 Vault policy 이름
- TTL : 로그인 유효 기간 (ex, 24h)
```
vault write auth/kubernetes/role/$ROLE_NAME \
    bound_service_account_names=$SERVICE_ACCOUNT_NAME_OF_POD \
    bound_service_account_namespaces=$POD_NAME_SPACE \
    policies=$POLICY_NAME \
    ttl=$TTL
```
8. Deployment 에 Vault agent 설정 추가
-  vault.hashicorp.com/role: 사용할 Vault auth role 이름을 정의한다.
- vault.hashicorp.com/agent-inject-secret-$PREFIX : 저장될 파일 경로 prefix를 표시한다. 
- 예를 들어 'agent-inject-secret-picup' 을 key로 하면 읽어온 데이터는 '/vault/secret/picup'에 저장된다.
- vault.hashicorp.com/agent-inject-secret 의 값은 읽을 vault secret path를 의미한다.
- vault.hashicorp.com/agent-inject-template-$PREFIX : 파일에 저장되는 형태를 결정한다.
- .Data 의 형태를 확인하고 저장할 포맷을 정의한다. 
- 예시의 .Data는 map[created:20240516 version:1] 꼴이었고, k=v 로 저장하는 Template 코드를 정의했다.
- source 로 secret 값이 저장된 파일을 환경 변수로 올린다.
``` yaml
ex)
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: vault-injector-test
  name: vault-injector-test
  namespace: picup
spec:
  replicas: 1
  selector:
    matchLabels:
      app: vault-injector-test
  template:
    metadata:
      annotations:
        vault.hashicorp.com/role: internal-app
        vault.hashicorp.com/agent-inject: "true"
        vault.hashicorp.com/agent-inject-secret-picup: picup/common
        vault.hashicorp.com/agent-inject-template-picup: |
            {{- with secret "picup/common" -}}
            {{- range $k, $v := .Data }}
            {{- $k }}={{ $v }}
            {{end}}{{end}}
      labels:
        app: vault-injector-test
    spec:
      containers:
        - image: alpine
          args:
            - "sh"
            - "-c"
            - "source /vault/secrets/picup && sleep 10000"
          imagePullPolicy: IfNotPresent
          name: vault-injector-test
          resources: {}
      terminationGracePeriodSeconds: 30
```

### Ref
- [Vault helm](https://github.com/hashicorp/vault-helm)
- [Hashicorp/k8s-injector](https://developer.hashicorp.com/vault/docs/platform/k8s/injector)
