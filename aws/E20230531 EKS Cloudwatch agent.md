# E20230531 EKS Cloudwatch agent.md

EKS의 Metric 정보를 Cloudwatch로 모니터링한다.

https://catalog.us-east-1.prod.workshops.aws/workshops/9c0aa9ab-90a9-44a6-abe1-8dff360ae428/ko-KR/90-monitoring/100-build-insight

## Fluentbit를 Daemonset으로 

#### cwagent-fluent-bit-quickstar 설치한다.

```
wget https://raw.githubusercontent.com/aws-samples/amazon-cloudwatch-container-insights/latest/k8s-deployment-manifest-templates/deployment-mode/daemonset/container-insights-monitoring/quickstart/cwagent-fluent-bit-quickstart.yaml
```

#### 환경 변수 선언한다.
```
ClusterName=${ClusterName}
RegionName=${RegionName}
FluentBitHttpPort={FluentBitHttpPort}
FluentBitReadFromHead='Off'
[[ ${FluentBitReadFromHead} = 'On' ]] && FluentBitReadFromTail='Off'|| FluentBitReadFromTail='On'
[[ -z ${FluentBitHttpPort} ]] && FluentBitHttpServer='Off' || FluentBitHttpServer='On'
```

```
sed -i -e 's/{{cluster_name}}/'${ClusterName}'/;s/{{region_name}}/'${RegionName}'/;s/{{http_server_toggle}}/"'${FluentBitHttpServer}'"/;s/{{http_server_port}}/"'${FluentBitHttpPort}'"/;s/{{read_from_head}}/"'${FluentBitReadFromHead}'"/;s/{{read_from_tail}}/"'${FluentBitReadFromTail}'"/' cwagent-fluent-bit-quickstart.yaml 
```

#### 해당 파일을 오픈한 후, fluent-bit의 DaemonSet의 spec으로 추가한다.

```
affinity:
  nodeAffinity:
    requiredDuringSchedulingIgnoredDuringExecution:
      nodeSelectorTerms:
      - matchExpressions:
        - key: eks.amazonaws.com/compute-type
          operator: NotIn
          values:
          - fargate
```

#### kubectl apply
```
kubectl create ns amazon-cloudwatch
kubectl apply -f cwagent-fluent-bit-quickstart.yaml 
```

## IAM policy 추가
아래 IAM policy를 노드 인스턴스에 추가한다.
```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Action": [
                "ec2:DescribeVolumes"
            ],
            "Resource": "*",
            "Effect": "Allow"
        }
    ]
}
```

## Could not retrieve ec2 metadata from IMDS    

Fluentbit pod에서 아래와 같은 IMDS 에러가 로그되는 경우이다.
```
[2023/05/30 01:42:47] [error] [filter:aws:aws.2] Could not retrieve ec2 metadata from IMDS   
```

해당 pod에 들어가 아래 커멘드를 입력한다.
```
curl -w "%{http_code}\n" http://169.254.169.254/
```

401 응답이 발생하는 경우 ec2 인스턴스에 접속하여 메타 데이터 옵션 -> IMDSv2를 optional로 수정해준다.

![image](https://github.com/ecsimsw/daily-note-public/assets/46060746/31bda50a-79c7-4713-a2c4-99af72376c36)

