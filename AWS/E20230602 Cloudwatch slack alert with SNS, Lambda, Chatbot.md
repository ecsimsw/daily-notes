# Cloudwatch metric alert 

<img width="756" alt="image" src="https://github.com/ecsimsw/daily-note-public/assets/46060746/ccd05d2b-a6ff-4540-ac6a-de8112a1c6d2">

cloudwatch 에서 수집한 metric 정보를 바탕으로 설정한 기준치 이상인 경우 알람을 전송할 수 있도록 한다.

</br>

## 1. Create SNS

### SNS → Topics → Create topic 

</br>

<img width="752" alt="image" src="https://github.com/ecsimsw/daily-note-public/assets/46060746/8b36bab3-3958-4f93-97e9-c1af1ea41c60">

</br>
</br>

## 2. Create cloudwatch alarm

Alarm을 발생시킬 데이터와 조건을 정의한다.

</br>

<img width="919" alt="image" src="https://github.com/ecsimsw/daily-note-public/assets/46060746/d9525bf4-427b-4d5f-8d53-725c3fbd8708">

</br>

## 3. Set trigger / Message sender

Topic에 생성된 alarm event를 받아 처리 (메시지 전송)할 sender를 설정한다.     

나는 Lambda로 외부 api 요청을 하는 방법이랑 Chatbot을 사용하는 방법 두가지를 테스트했다. 
일단 그 두 방식을 모두 적긴 하는데 Lambda는 좀 더 유연하게 메시지를 만들 수 있지만 lambda 코드를 관리하는게 불편하고 그냥 Chatbot으로 설정하기로 했다.    

아 참고로 비용은 SNS - HTTP/HTTPS로 전송하는 경우 매월 첫 1백만 개는 무료라 거의 생각을 안해도 될 정도로 싸다. Chatbot 역시 다른 비용이 추가되지 않는다. 
생각해야 할 것은 Lambda로 전송하는 경우의 Lambda 비용과 Cloudwatch의 metric/alarm 비용인데 특히 후자가 생각보다 비싸니 확인하길 바란다.

</br>

## 3-1. AWS chatbot을 사용하는 방법

### AWS chatbot → Configure new client

</br>

<img width="760" alt="image" src="https://github.com/ecsimsw/daily-note-public/assets/46060746/77c1d700-c24d-4037-b2cd-5134d1edb060">


### configure new channel → configure slack channel → set channel name 

채널 정보와 topic 정보를 입력한다. Channel guardrail policies에 cloudwatchReadOnlyAccess를 추가한다.

</br>

<img width="768" alt="image" src="https://github.com/ecsimsw/daily-note-public/assets/46060746/0cb8af81-4a1c-465f-9dd6-79956c988902">

</br>

### Result

</br>

<img width="928" alt="image" src="https://github.com/ecsimsw/daily-note-public/assets/46060746/21cb6504-b35e-41c7-bf76-443160ba2110">

</br>
</br>

## 3-2 Lambda를 사용하는 방법

### Lambda 생성

<img width="961" alt="image" src="https://github.com/ecsimsw/daily-note-public/assets/46060746/1e265e46-b373-4d77-bdc4-98329b9f9be1">

</br>

python으로 runtime을 잡고 아래 lambda_function.py를 code source로 정의한다.

``` python
import urllib3
import json
http = urllib3.PoolManager()
def lambda_handler(event, context):
    url = "${URL}"
    message = json.loads(event['Records'][0]['Sns']['Message'])

    alarm_name = message['AlarmName']
    new_state = message['NewStateValue']
    state_changed_time = message['StateChangeTime']
    new_state_reason = message['NewStateReason']

    msg = {
        "channel": "${CHANNEL_NAME}",
        "username": "cloudwatch",
        "icon_emoji" : "",
        "text": (
            "time : " + state_changed_time + "\n"
            "alarm name : " +  alarm_name + "\n" \
            "state : " + new_state + "\n" \
            "reason : " + new_state_reason + "\n"
        )
    }
    encoded_msg = json.dumps(msg).encode('utf-8')
    resp = http.request('POST',url, body=encoded_msg)
    print({
        "message": event['Records'][0]['Sns']['Message'],
        "status_code": resp.status,
        "response": resp.data
    })
```

</br>

###  Cloudwatch alarm event sample
Test 이벤트를 정의할 때 도움이 되었던 Cloudwatch alarm sample이다.

``` json
{
  "Records": [
    {
      "EventSource": "aws:sns",
      "EventVersion": "1.0",
      "EventSubscriptionArn": "arn:aws:sns:eu-west-1:000000000000:cloudwatch-alarms:xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
      "Sns": {
        "Type": "Notification",
        "MessageId": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
        "TopicArn": "arn:aws:sns:eu-west-1:000000000000:cloudwatch-alarms",
        "Subject": "ALARM: \"Example alarm name\" in EU - Ireland",
        "Message": "{\"AlarmName\":\"Example alarm name\",\"AlarmDescription\":\"Example alarm description.\",\"AWSAccountId\":\"000000000000\",\"NewStateValue\":\"ALARM\",\"NewStateReason\":\"Threshold Crossed: 1 datapoint (10.0) was greater than or equal to the threshold (1.0).\",\"StateChangeTime\":\"2017-01-12T16:30:42.236+0000\",\"Region\":\"EU - Ireland\",\"OldStateValue\":\"OK\",\"Trigger\":{\"MetricName\":\"DeliveryErrors\",\"Namespace\":\"ExampleNamespace\",\"Statistic\":\"SUM\",\"Unit\":null,\"Dimensions\":[],\"Period\":300,\"EvaluationPeriods\":1,\"ComparisonOperator\":\"GreaterThanOrEqualToThreshold\",\"Threshold\":1.0}}",
        "Timestamp": "2017-01-12T16:30:42.318Z",
        "SignatureVersion": "1",
        "Signature": "Cg==",
        "SigningCertUrl": "https://sns.eu-west-1.amazonaws.com/SimpleNotificationService-xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx.pem",
        "UnsubscribeUrl": "https://sns.eu-west-1.amazonaws.com/?Action=Unsubscribe&SubscriptionArn=arn:aws:sns:eu-west-1:000000000000:cloudwatch-alarms:xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
        "MessageAttributes": {}
      }
    }
  ]
}
```

</br>

### Set trigger

그리고 이 Lambda가 실행될 trigger로 SNS를 붙여준다.

</br>

<img width="809" alt="image" src="https://github.com/ecsimsw/daily-note-public/assets/46060746/f0a9e383-5137-4170-916b-c06edd9d58d1">

</br>

### Result

<img width="713" alt="image" src="https://github.com/ecsimsw/daily-note-public/assets/46060746/c1f0e39b-2685-43cc-b98c-615005eb34e3">

