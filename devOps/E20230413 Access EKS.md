
## Access EKS

여러 AWS 계정을 사용, On premise 서버를 하고 있는 상황서 

<br>

### 1. AWS IAM Access key

1-0 Policy

가장 먼저는 해당 IAM user가 EKS 접속 권한이 있는지 확인하고 없다면 Role을 추가한다.

1-1 AWS IAM Access key 발급

IAM role → Users → SecurityCredentials → Access key

![image](https://user-images.githubusercontent.com/46060746/232652247-3d9c02dd-5355-4a90-b517-a01ae222b972.png)


1-2 Create new Access Key
- create access key 
- check 'Access key best practices & alternatives' (CLI)
- copy 'Access key and Secret access key'

<br>

## 2. AWS CLI

2-1 install cli tool

```
brew install awscli
```

2-2 aws configure of each profile

```
# default 프로파일 등록
aws configure

# 지정된 이름의 프로파일 등록
aws configure --profile <PROFILE_NAME>
```

각 계정을 지칭할 profile_name을 정하고 각 aws profile 을 등록한다. 


2-3 created files

위 aws configure로 여러 profile을 등록했다면 아래와 같이 두개의 파일이 만들어진다.

#### ~/.aws/credentials

```
[default]
aws_access_key_id = AXXXXXXXXXXXXXZ
aws_secret_access_key = 8XXXXXXXXXXXXXXXXXXXXXXXXXXXp

[prod-ap1]
aws_access_key_id = AXXXXXXXXXXXXXA
aws_secret_access_key = BXXXXXXXXXXXXXXXXXXXXXXXXXXXE

[prod-ap2]
aws_access_key_id = AXXXXXXXXXXXXXA
aws_secret_access_key = BXXXXXXXXXXXXXXXXXXXXXXXXXXXE

[dev-4-ap1]
aws_access_key_id = AXXXXXXXXXXXXXS
aws_secret_access_key = jXXXXXXXXXXXXXXXXXXXXXXXXXXXC

[dev-4-ap2]
aws_access_key_id = AXXXXXXXXXXXXXZ
aws_secret_access_key = jXXXXXXXXXXXXXXXXXXXXXXXXXXXC

[dev-7-ap1]
aws_access_key_id = AXXXXXXXXXXXXXZ
aws_secret_access_key = 8XXXXXXXXXXXXXXXXXXXXXXXXXXXp

[dev-7-ap2]
aws_access_key_id = AXXXXXXXXXXXXXZ
aws_secret_access_key = 8XXXXXXXXXXXXXXXXXXXXXXXXXXXp
```

#### ~/.aws/config

```
[default]
region = ap-northeast-2
output = json

[profile prod-ap1]
region = ap-northeast-1
output = json

[profile prod-ap2]
region = ap-northeast-2
output = json

[profile dev-4-ap1]
region = ap-northeast-1
output = json

[profile dev-4-ap2]
region = ap-northeast-2
output = json

[profile dev-7-ap1]
region = ap-northeast-1
output = json

[profile dev-7-ap2]
region = ap-northeast-2
output = json
```

<br>

## 3. EKS access

EKS 접속 시에는 다음과 같이 할 수 있다.

3-1 export aws profile want to use

```
export AWS_PROFILE={PROFILE_NAME} 
```

3-2 update kubeconfig

AWS CLI 툴에서 접근하고자 하는 클러스터 정보에 따라 kubeConfig 파일을 수정한다. ~/.kube/config 안에는 클러스터 정보, 사용자 정보, 클러스터에 어떤 사용자로 접근할 것인지를 나타내는 Context 가 저장되어 있다. 

```
 aws eks --region {REGION_CODE} update-kubeconfig --name {CLUSTER_NAME}
```

#### ~/.kube/config
```
apiVersion: v1
clusters:
- cluster:
    certificate-authority-data: LSXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX==
    server: https://XXXXXXXXXXXXXXXXXXXXXXXX.gr7.ap-northeast-2.eks.amazonaws.com
  name: arn:aws:eks:ap-northeast-2:XXXXXXXXXXXXXXXXXX:cluster/{CLUSTER_NAME}
contexts:
- context:
    cluster: arn:aws:eks:ap-northeast-2:XXXXXXXXXXXXXXXXXX:cluster/{CLUSTER_NAME}
    user: arn:aws:eks:ap-northeast-2:XXXXXXXXXXXXXXXXXX:cluster/{CLUSTER_NAME}
  name: arn:aws:eks:ap-northeast-2:XXXXXXXXXXXXXXXXXX:cluster/{CLUSTER_NAME}
current-context: arn:aws:eks:ap-northeast-2:XXXXXXXXXXXXXXXXXX:cluster/{CLUSTER_NAME}
kind: Config
preferences: {}
users:
- name: arn:aws:eks:ap-northeast-2:XXXXXXXXXXXXXXXXXX:cluster/{CLUSTER_NAME}
  user:
    exec:
      apiVersion: client.authentication.k8s.io/v1beta1
      args:
      command: aws
      env:
      - name: AWS_PROFILE
        value: dev-7-ap1
```

aws eks 명령어는 환경 변수로 설정된 AWS_PROFILE을 읽어 어떤 aws profile access key를 사용할 것인지를 확인하고 접속하고자 하는 사용자가 해당 클러터스에 접근이 가능한지를 확인한다. (이때 아마 eks:DescribeCluster 라는 role이 필요할 것이다)

만약 맞다면 cluster 정보를 kubeConfig에 저장한다. 현재 사용 중인 cluster 정보는 kubeConfig 파일에 'current-context'를 키로 저장되는데 aws eks update-kubeconfig는 클러스터 정보 저장과 동시에 'current-context'의 값을 해당 context로 바꿔 사용자가 어떤 클러스터 접속 중인지 확인하는 것이다.

<br>

## 4. EKS cluster permission

4-1 Add user kubernetes cluster ConfigMap 'aws-auth'

EKS 클러스터에서도 user에 접근 허가를 줘야 한다. 관리자 계정으로 cluster에 접근하여 'aws-auth' 라는 이름의 configmap을 수정한다. data.mapUsers에 아래와 같은 포맷으로 접근하고자 하는 user를 입력해준다.

```
│ apiVersion: v1                                                                           
│ data:                                                                                
│   mapUsers:                                                                                                                                       
│     - groups:                                                                            
│       - system:masters                                                                   
│       userarn: arn:aws:iam::{account-id}:user/{username}                                   
│       username: {username} 
```

