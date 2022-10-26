## Apply SonarScanner on Jenkins 

### SonarScanner
sonarqube는 코드를 분석해서 버그 위험 여부, 보안 취약점, 코드 스멜, 코드 중복 등의 코드 분석 툴이다. 이를 CI 프로세스에 적용하여 정상 빌드된 코드를 스캔하고 SonarQube를 업데이트 할 생각이다. 이를 위해 필요한 프로그램이 SonarScanner이다.   

사실 SonarQube 플랫폼을 설치해서 SonarQube를 위한 서버를 직접 띄우면 각 빌드 환경에 대한 SonnarScanner가 함께 설치된다. 내 상황의 경우 SonnarCloud를 사용했기 때문에 직접 빌드 환경에 맞는 SonnarScanner를 설치하고 이를 실행 시켜야 했다.

<br>

### SonarScanner for XXX
각 빌드 환경 (빌드툴)에 맞는 SonarScanner가 각각 제공된다. 이를 테면 Gradle 일 경우 `SonarScanner for Gradle` 를 사용할 수 있고, 그럴 경우 플러그 인만 간단히 추가하는 것으로 Gradle이 제공하는 함수로 간단히 SonarQube Scanner 실행이 가능하다.


### Script

```
stage('sonarqube') {
    SONAR_SCANNER_VERSION = "4.7.0.2747"
    SONAR_SCANNER_PATH = "$HOME/.sonar/sonar-scanner-$SONAR_SCANNER_VERSION-linux/bin/"

    SONAR_PROJECT_ORGANIZATION = ${SONAR_PROJECT_ORAGANIZATION}
    SONAR_PROJECT_KEY = ${SONAR_PROJECT_NAME}
    SONAR_TOKEN = ${SONAR_PROJECT_TOKEN}
    SCANNING_PROJECT_PATH = "api"

    sh """
        curl --create-dirs -sSLo $HOME/.sonar/sonar-scanner.zip https://binaries.sonarsource.com/Distribution/sonar-scanner-cli/sonar-scanner-cli-$SONAR_SCANNER_VERSION-linux.zip 
        apt-get update && apt-get install unzip
        unzip -o $HOME/.sonar/sonar-scanner.zip -d $HOME/.sonar/
    """

    sh """
      cd $SCANNING_PROJECT_PATH
      $SONAR_SCANNER_PATH/sonar-scanner \
      -Dsonar.organization=$SONAR_PROJECT_ORGANIZATION \
      -Dsonar.projectKey=$SONAR_PROJECT_KEY \
      -Dsonar.sources=. \
      -Dsonar.host.url=https://sonarcloud.io   \
      -Dsonar.login=$SONAR_TOKEN
    """
}
```
 
 
 ![image](https://user-images.githubusercontent.com/46060746/197998021-fe28b2c3-222e-444f-86af-725c528b8b05.png)

![image](https://user-images.githubusercontent.com/46060746/197998167-7e0aed4a-3d13-48d5-8727-48643443613f.png)


