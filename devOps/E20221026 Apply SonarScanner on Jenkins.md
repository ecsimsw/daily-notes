
# Apply SonarScanner on Jenkins 

<br>

## SonarScanner
sonarqube는 코드를 분석해서 버그 위험 여부, 보안 취약점, 코드 스멜, 코드 중복 등의 코드 분석 툴이다. 이를 CI 프로세스에 적용하여 정상 빌드된 코드를 스캔하고 SonarQube를 업데이트 할 생각이다. SonarScanner는 코드를 분석한다.    

사실 SonarQube 플랫폼을 설치해서 SonarQube를 위한 서버를 직접 띄우면 각 빌드 환경에 대한 SonnarScanner가 함께 설치된다. 내 상황의 경우 SonnarCloud를 사용했기 때문에, 그리고 NPM, Gradle 등 외 빌드툴에 구애받지 않는 코드 분석을 원했기 때문에, 직접 빌드 환경에 맞는 SonnarScanner를 설치하고 이를 실행 시켜야 했다.

<br>

## SonarScanner for XXX
각 빌드 환경 (빌드툴)에 맞는 SonarScanner가 각각 제공된다. 이를 테면 Gradle 일 경우 `SonarScanner for Gradle` 를 사용할 수 있고, 그럴 경우 플러그인만 간단히 추가하는 것으로 Gradle이 제공하는 함수로 간단히 SonarQube Scanner 실행이 가능하다.

<br>

## Apply SonarScanner on Jenkins 

SonarScanner를 직접 설치해서 구동하는 방법은 간단하다. 예제 스크립트들은 내가 설정했던 젠킨스의 코드를 예시로 한다. 젠킨스에서 제공하는 환경 변수 등은 직접 맞춰줘야 한다. 설정은 다음 [레퍼런스](https://docs.sonarqube.org/latest/analysis/scan/sonarscanner/)를 참고 하였다. 

`ex) $HOME -> /Desktop`



<br>

### 1. SonarScanner를 설치한다.

환경에 curl과 unzip이 설치되어 있는 상태에서 아래 명령어를 수행한다.

```
curl --create-dirs -sSLo $HOME/.sonar/sonar-scanner.zip https://binaries.sonarsource.com/Distribution/sonar-scanner-cli/sonar-scanner-cli-4.7.0.2747-linux.zip 
unzip -o $HOME/.sonar/sonar-scanner.zip -d $HOME/.sonar/
```

<br>

### 2. Project key, token, organization을 확인한다.

SonarCloud에서 프로젝트에 해당하는 `SONAR_PROJECT_ORAGANIZATION`, `PROJECT_NAME` , `PROJECT_TOKEN` 을 확인할 수 있다.

<img width="456" alt="image" src="https://user-images.githubusercontent.com/46060746/199001077-523e8bba-2856-44e4-a5a8-ede82d1ad8bc.png">

```
SONAR_PROJECT_ORGANIZATION = ${SONAR_PROJECT_ORAGANIZATION}
SONAR_PROJECT_KEY = ${SONAR_PROJECT_NAME}
SONAR_TOKEN = ${SONAR_PROJECT_TOKEN}
```

<br>

### 3. SonarScanner 실행한다.

코드 분석을 진행할 폴더에서 sonar-scanner를 실행한다. sonar-scanner는 앞서 설치한 `sonar-scanner.zip`을 압축 해제하고 `sonar-scanner-$SONAR_SCANNER_VERSION-linux/bin/` 아래에 위치한다.

```
sh "cd $SCANNING_PROJECT_PATH"
sh """
  $SONAR_SCANNER_PATH/sonar-scanner \
  -Dsonar.organization=$SONAR_PROJECT_ORGANIZATION \
  -Dsonar.projectKey=$SONAR_PROJECT_KEY \
  -Dsonar.sources=. \
  -Dsonar.host.url=https://sonarcloud.io   \
  -Dsonar.login=$SONAR_TOKEN
"""
```

프로그램 인자로 -Dsonar.${KEY} = ${VALUE} 꼴로 인자를 줄 수 있다. 아래 예시에서 KEY-VALUE를 확인하여 앞서 확인한 Project의 key 값을 입력해준다.

<img width="841" alt="image" src="https://user-images.githubusercontent.com/46060746/199000458-db4eba64-05f1-46f8-b282-9e3747a380b3.png">


<br>

### 4. (Optional) 프로젝트 브랜치 이름을 설정한다.

Sonar-scanner를 실행할 때 SonarCloud의 프로젝트에 브랜치 이름으로 표시될 브랜치 이름을 직접 입력할 수 있다. 아래 인자를 실행 시 추가하면 된다. 입력하지 않을 경우, `Main Branch` 로 기본 값이 설정된다.
 
`-Dsonar.branch.name=$BRANCH_NAME`

아래는 기본 값으로 분석한 결과와 test라는 이름으로 직접 branch 명을 지정했을 때의 sonarCloud에서 표시되는 방식을 보여주는 스크린샷이다.

![IMG_1996](https://user-images.githubusercontent.com/46060746/199001476-aef068bf-1721-49c8-b7c5-a2a721106d06.jpg)


<br>

## 더 나아가서 

이 앞선 과정을 젠킨스 build 과정에 추가하였다. build가 성공적으로 되는 파이프라인의 경우 그 이후에 SonarScanner를 실행하는 것이다.

<br>

### 1. Docker image로 스캔 환경 구성하기

우리 팀 환경의 경우 스캔을 위해 매번 수행하는 작업들이 많았다. `curl`, `unzip` 패키지를 설치했어야 하고, `sonar-scanner.zip` 을 매번 설치하여, 압축을 해제해야 한다.    

팀의 젠킨스는 kubernetes plugin을 이용하여 원하는 step(빌드 과정)이 실행되길 원하는 컨테이너를 지정할 수 있었다. 해당 과정을 매번 반복하지 않을 수 있도록 컨테이너 이미지를 만들었다. 해당 이미지의 dockerfile은 다음과 같다. 

```
FROM ubuntu:20.04

RUN apt-get -y update \
	&& apt-get -y install curl \
	&& apt-get -y install unzip

RUN curl -sL https://deb.nodesource.com/setup_14.x | bash - \
	&& apt-get -y install nodejs

RUN curl --create-dirs -sSLo /sonarqube/sonar-scanner.zip https://binaries.sonarsource.com/Distribution/sonar-scanner-cli/sonar-scanner-cli-4.7.0.2747-linux.zip
RUN unzip -o /sonarqube/sonar-scanner.zip -d /sonarqube/
```

이 컨테이너에서 스캔할 코드가 node.js 코드여서 그런건지 `node`를  필요로 했다. 참고하되 본인 환경에 맞춰 수정해야 할 것이다.

<br>

### 2. 병렬 처리로 시간 단축

코드를 스캔하는데 시간이 생각보다 오래 걸린다. 우리 팀의 환경에선 총 658개의 코드 파일 탐색했고, 이에 3분 전후의 시간이 걸렸다. 이를 젠킨스의 병렬 처리로 해결할 수 있었다. 

해당 프로젝트의 큰 파이프라인 프로세스는 다음과 같았다,

```
1. git clone
2. project build
3. test
4. make docker image and push
5. k8s roll out
```

우선 build가 안되는 코드는 분석의 의미가 없다고 생각했다. 생각보다 build에서 깨지는 코드가 많았어서 build 이후에 코드 분석을 진행하는게 좋겠다고 생각했고, test와 docker image로 만들어 push 하는 과정이 대략 sonar-scanner의 스캔 시간과 비슷하여 이 둘을 합쳐 병렬 처리할 수 있도록 만들었다. 

대략 구조는 다음과 같다.

```
parallel {
   stage('test and ECR') {
      stage('test') {}
      stage('push to ECR') {}
   }
   stage('sonar-scanner') {}
}
```

그래도 스캐닝 시간이 좀 더 오래 걸려서 약 2분의 시간을 단축할 수 있었다.
