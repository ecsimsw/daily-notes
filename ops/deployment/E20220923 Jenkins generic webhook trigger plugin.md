## Jenkins generic webhook trigger plugin

### 0. What I wanted to do → trigger filtering by tag name

한 레포지토리 안에서, branch나 comment가 아닌, tag 이름의 포맷으로 구분된 jenkins job이 있다. (1개 repository에 여러 Job이 물려있는 경우)    
webhook request 안의 정보를 깔끔하게 이용할 수 있는 방법을 고민했고, 괜찮은 플러그인을 찾아 소개하게 되었다.

<br>

### 1. Generic Webhook Trigger Plugin 

[generic-webhook-trigger](https://plugins.jenkins.io/generic-webhook-trigger) 을 설치한다.

<br>

### 2. 원하는 정보에 해당하는 키 값 찾기
webhook의 요청이 어떻게 나가는지 확인해본다. (eg, [Gitlab-webhook](https://docs.gitlab.com/ee/user/project/integrations/webhook_events.html))    
예를 들면, 웹 훅의 키 ref에 해당하는 값을 가져와 branch 정보나 Tag 정보를 가져 올 수 있겠다는 것을 확인할 수 있다.

![Unknown](https://user-images.githubusercontent.com/46060746/191938054-fcc2357a-3cf1-40e9-92bd-346b09297833.png)

<br>

### 3. Jenkins job configuration → Build triggers

원하는 Job의 Build triggers에서 Generic webhook trigger 사용을 체크한다.


<img width="677" alt="image" src="https://github.com/ecsimsw/daily-note-public/assets/46060746/b50af0ee-3dfa-4b20-a221-c3dc9046ca32">


<br>


### 4. Jenkins contetnt parameters setting

Variable name → 원하는 변수명 (ex, ref)   
Expression → 원하는 값의 key (ex, $.ref)


![Unknown](https://user-images.githubusercontent.com/46060746/191938539-bc56c968-b79f-41cf-a5bb-5bc62a05765e.png)


<br>


### 5. Set token / Check invoke url

원하는 토큰 이름을 입력한다. 


<img width="888" alt="image" src="https://github.com/ecsimsw/daily-note-public/assets/46060746/d42e12e5-c3a5-4c05-8f98-befde0f28579">



Webhook의 URL이 이 토큰을 파라미터로 갖는다.

- Query parameter /invoke?token=TOKEN_HERE
- A token header token: TOKEN_HERE
- A Authorization: Bearer header Authorization: Bearer TOKEN_HERE


<br>

기본 invoke url은 http://${JENKINS_URL}/generic-webhook-trigger/invoke이다. 

아래는 쿼리파라미터로 토큰을 표시한다고 하면 invoke url의 예시이다.   

ex) 
JENKINS URL : http://ecsimsw.jenkins   
TOKEN : my-token   
→ invoke url → http://ecsimsw.jenkins/generic-webhook-trigger/invoke?token=my-token 


<br>


### 6. Configure Webhook

Webhook을 설정한다. Webhook을 발생시킬 이벤트 종류를 선택하고, 타겟 Url을 앞서 확인한 invoke url로 하면 된다.

이때 hook의 content type을 application/json으로 한다.


설정 후 해당 이벤트를 발생시켰을 때 HTTP response status가 200이면서 아래와 같이 Jenkins에 설정한 트리거에 대한 정보가 반환된다면 알맞게 Webhook을 발생시켰고, Jenkins Trigger가 알맞게 응답한 것이다.


<img width="868" alt="image" src="https://github.com/ecsimsw/daily-note-public/assets/46060746/7401e0a4-67d8-4b75-8432-75af169dd4c5">



만약 응답코드가 200이 아니거나 응답 본문이 없다면 잘못된 것이다. 설정을 다시 확인한다.

응답코드가 200이면서 Job에 대한 정보가 올바른데 빌드 실행이 안되었다면 Trigger는 Webhook을 받을 수 있으나 실행 조건이 맞지 않아서이다. 특히 Optional filter에 다른 값이 들어가진 않았는지, 정규표현식을 잘못쓰진 않았는지 다시 확인한다.


![1](https://user-images.githubusercontent.com/46060746/191939255-f5eb52b7-9388-4936-9f43-48938b07cd52.png)



<br>



### 7. Optional filter

This is an optional feature. If specified, this job will only trigger when given expression matches given text.



Webhook의 요청 본문의 정보로 빌드 여부를 쉽게 필터링할 수 있다. 처음 요구 사항처럼 tag 이름이나 branch 정보를 받아 필터링할 것이다.



아래의 예시는 Webhook의 ref라는 변수로 정의한 값이 `refs/tags/core-v*`에 만족하는지 여부를 확인하는 필터링이다. $ref는 앞서 Post content parameter에서 요청 본문의 ref 값을 ref라는 이름으로 변수로 만들어둔 것을 사용한 것이다. 즉 태그 발생 이벤트에 Webhook을 걸고, 해당 hook의 본문 값을 읽어 발생된 태그가 core-v*임을 확인한다.

![2](https://user-images.githubusercontent.com/46060746/191939293-be28457b-2523-48fa-a4de-1a99f465361f.png)
