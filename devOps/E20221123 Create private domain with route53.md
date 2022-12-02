## How to add new hosted VPC

회사 내부에서 사용하는 서비스 도메인이 public하게 열려있었다. 이를 Route53의 private dns를 사용해서 지정된 VPC 안에서만 접근할 수 있는 내부용 dns를 만들어 사용할 수 있도록 하였다.     
    
이때 회사에서 사용하는 AWS 계정이 많았고, 접근해야하는 VPC가 각 계정마다 따로 있어, private dns를 전면에 두고 다른 계정 VPC에 권한 요청/허용하는 것으로 각 VPC를 등록해야 했다.   

![image](https://user-images.githubusercontent.com/46060746/205311423-9123467c-7ef5-4cd4-9d2e-abefd5713465.png)

<br>

## How to configure
Route53에서 Dns를 생성할 때 private hosted zone 를 선택할 수 있다.    
이때 같은 도메인 이름으로 public hosted zone이 이미 존재해도 상관없다. 예를 들면 public host zone 'ecsimsw.app'가 이미 존재해도 private host zone 'ecsimsw.app'를 만들 수 있는 것이다.    

![image](https://user-images.githubusercontent.com/46060746/205311859-7d2820c4-cf2d-4752-8cc3-4f8d4eb64129.png)

<br>

## Attach VPC to be used

접근할 수 있는 VPC를 선택한다.

![image](https://user-images.githubusercontent.com/46060746/205311747-9f82ff13-64d7-4a00-8810-c54ba074ee9d.png)


이때 계정 밖의 VPC를 등록하려고 하면 이런 에러가 출력된다.

![image](https://user-images.githubusercontent.com/46060746/205312015-0fc1f7ca-58ae-438f-9789-f2bfa7d2c7f8.png)

<br>

## Route53-private-hosted-zone

Route53을 설정하는 계정 밖의 VPC를 hosted zoned으로 등록하기 위한 과정이다.   
AWS CLI로 두 계정에 접근할 수 있어야 한다. route53을 설정하는 계정을 Host account, 접근하려는 VPC의 계정을 Client account라고 칭하겠다,

### Host account : create association authorization 
Host account에서 Client account에 vpc association 허가를 요청한다.

```
export AWS_PROFILE=<host_profile_name>
aws route53 create-vpc-association-authorization --hosted-zone-id <hosted-zone-id> --vpc VPCRegion=<region>,VPCId=<vpc-id> --region <region>
```
 
### Client account : associate vpc with hosted zone 
Client account에서 요청을 확인한다.

```
export AWS_PROFILE=<client_profile_name>
aws route53 associate-vpc-with-hosted-zone --hosted-zone-id <hosted-zone-id> --vpc VPCRegion=<region>,VPCId=<vpc-id> --region <region>
```
 
### Host account : delete association
Association을 삭제하려면 Host account에서 다음 명령어를 수행한다. 사용이 끝난 vpc association은 제거하는걸 추천한다.   

```
export AWS_PROFILE=<host_profile_name>
aws route53 delete-vpc-association-authorization --hosted-zone-id <hosted-zone-id> --vpc VPCRegion=<region>,VPCId=<vpc-id> --region <region>
```

Ref, https://aws.amazon.com/ko/premiumsupport/knowledge-center/route53-private-hosted-zone/

<br>

## Check hosted zone
 
Hosted zone에 VPC가 추가된 것이 확인되면 성공이다. 
 
![image](https://user-images.githubusercontent.com/46060746/205316606-de2d03bc-e9aa-4628-8f58-d76350c8dbde.png)
