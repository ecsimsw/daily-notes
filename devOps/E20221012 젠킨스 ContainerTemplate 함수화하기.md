## 젠킨스 ContainerTemplate 함수화하기

회사에서 젠킨스 YAML 스크립트로 PodTemplate를 정의하는데, template 내용을 좀 더 커스텀하게 다룰 수 있도록 함수화하는 작업을 맡았다.   
 
### 요구 사항 
 
회사에선 아래와 같은 형태의 YAML로 Template을 다루고 있었다.

``` yaml
podTemplate(yaml: '''
    apiVersion: v1
    kind: Pod
    spec:
      containers:
      - name: maven
        image: maven:3.8.1-jdk-8
        command:
        - sleep
        args:
        - 99d
      - name: golang
        image: golang:1.16.5
        command:
        - sleep
        args:
        - 99d
''') {
  node(POD_LABEL) {
    stage('Get a Maven project') {
      git 'https://github.com/jenkinsci/kubernetes-plugin.git'
      container('maven') {
        stage('Build a Maven project') {
          sh 'mvn -B -ntp clean install'
        }
      }
    }

    stage('Get a Golang project') {
      git url: 'https://github.com/hashicorp/terraform-provider-google.git', branch: 'main'
      container('golang') {
        stage('Build a Go project') {
          sh '''
            mkdir -p /go/src/github.com/hashicorp
            ln -s `pwd` /go/src/github.com/hashicorp/terraform
            cd /go/src/github.com/hashicorp/terraform && make
          '''
        }
      }
    }

  }
}
```

</br>

사실 [젠킨스 쿠버네티스 플러그인](https://github.com/jenkinsci/kubernetes-plugin)은 아래와 같은 형태로 함수로 containerTemplate를 정의하는 방식을 지원하고 있다. 
문제가 됐던 포인트는 해당 함수에서 파라미터로 지원하고 있는 필드가 매우 적어, 필요한 정보(ex, resources)를 누락해야 했다.

</br>

```
podTemplate {
    containerTemplate(name: 'maven', image: 'maven:3.8.1-jdk-8', command: 'sleep', args: '99d'),
    containerTemplate(name: 'golang', image: 'golang:1.16.5', command: 'sleep', args: '99d')
}
```

</br>

그래서 위 젠킨스 공식 꼴에 맞추되 필드에 사용자 자유를 주고 싶었고, groovy 문법으로 쓰여진 객체로 필드를 명시하면 이를 yaml로 바꿔 template으로 사용할 수 있는 방법을 고민하게 되었다.   

즉 원했던 사용 꼴은 다음과 같다. 주입하는 필드는 해당 정보가 적용되고, 주입하지 않는 필드는 기본 값을 따르도록 한다.

</br>

```
dockerTemplate(containers: [
    containerTemplate([name : "container_name", image: "image_info", resources : [limits : [cpu: "cpu_info", memory : "memory_info"]]])
])
```

</br>

### containerTemplate() : 필드 기본 값 처리

containerTemplate안에서 필수적인 필드 값의 기본 값을 주입하도록 했다. 인자로 주입한 container 정보에서, 우리 시스템에 꼭 필요한 필수 필드가 빠져있는 경우 디폴트 값을 주입한다. 이를 테면 컨테이너 이름, 이미지 정보, 리소스 제한 등이 필수 데이터였다.    

</br>

```
static def containerTemplate(def container) {
    container['name'] =  container['name'] ?: "DEFAULT_NAME"
    container['image'] = container['image'] ?: "DEFAULT_IMAGE"
    container['resources'] = container['resources'] ?: [limits : [cpu : "500m", memory : "1Gi"]]
    container['command']  = container['command'] ?: ["sleep"]
    container['args'] = container['args'] ?: ["infinity"]
    ...
    
    return container
}
```

</br>

### Snakeyaml : 객체를 yaml로 변환
이제 이 객체를 yaml로 만들어 기존 podTemplate yaml 값 안에 끼어 넣을 수 있도록 했다. yaml로의 변환은 snakeyaml 라이브러리를 이용했다. 

``` javascript
def dockerTemplate(def containers) {
  def containersYaml = new Yaml(options).dump(containers)
  def podTemplate = """
api version: v1
kind: Pod
spec:
  nodeSelector:
    ...
  tolerations:
    ...
  ${containersYaml}
  volumes:
   ...
 """
 return podTemplate
```

이렇게 되면 기존 podTemplate과 인덴트가 안맞는다. containers로 만든 yaml은 인덴트의 시작이 0이기 때문에 아래 출력에서 containers 아래의 name은 containers보다 인덴트가 더 낮은 것을 볼 수 있다. 원래는 들여 써 있어야한다.    

``` javascript
api version: v1
kind: Pod
spec:
  nodeSelector:
    ...
  tolerations:
    ...
  containers:
- name: DEFAULT_NAME
  image: DEFAULT_IMAGE_INFO
  resources:
  resources:
    limits: {cpu: 60m, memory: 1Gi}
  ...
```

resources 아래 limits를 표시한 스타일도 원하는 BLOCK 형식이 아닌 Basic Flow style이라는 JSON 모양의 yaml로 출력됨을 확인할 수 있다.

</br>

![image](https://user-images.githubusercontent.com/46060746/195314290-4da5dd74-35d2-4266-bbde-189c2f5f1787.png)

</br>

### DumperOptions : 인덴트 옵션 설정

이를 위해 'DumperOptions'을 사용했다. 한 인덴트 안에 space가 몇개인지, FlowStyle과 시작 인덴트 값이 몇인지를 지정한다. 

``` javascript
DumperOptions options = new DumperOptions();
options.setIndent(2);
options.setDefaultFlowStyle(DumperOptions.FlowStyle.BLOCK);
options.setIndicatorIndent(2);
options.setIndentWithIndicator(true);

def containersYaml = new Yaml(options).dump(containers)
```

이 옵션을 추가하여 변환된 Yaml은 의도한 바대로 시작 인덴트, Block 모양의 FlowStyle, 인덴트 space값이 적용됨을 확인할 수 있다.

``` javascript
api version: v1
kind: Pod
spec:
  nodeSelector:
    ...
  tolerations:
    ... 
  containers:
  - name: DEFAULT_NAME
    image: DEFAULT_IMAGE_INFO
    resources:
      limits:
        cpu: 60m
        memory: 1Gi
  volumes:
    ...
```
