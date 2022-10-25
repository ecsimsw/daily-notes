## Fluentd에서 Filebeats + Logstash로 이전
팀에서 요청 ip를 기준으로 사용자의 위치 분포를 확인하고 이를 Kibana에 올려 모니터링하고 싶었다. 그 과정에서 기존에는 애플리케이션의 로그 수집과 전처리를 위해 Fluentd를 사용했는데, 이번에 Filebeats + Logstash로 바꾸게 되었다.

</br>

### Filebeats와 Logstash
가장 우선은 Fluentd에서 원하는 플러그인(Geolocation IP) 사용에 문제가 있었다. 그리고 Logstash로 전환 후 바로 적용이 가능했다. 

사실 그게 가장 큰 이유가 되었다. Logstash는 ELK stack의 일부다. 어쨌든 Elasticsearch랑 함께 사용하기에 Logstash가 더 편한 것도 사실이고, 그걸 배제하더라도 Fluentd를 사용할 때 생기는 문제는 내 설정과 Elasticsearch와의 호환을 모두 고민하게 된다면, Logstash에서는 일단 호환은 덜 생각하고 설정을 의심할 수 있게 되었다. 한마디로 ELK 스택 안에 함께한다는 사실만으로 Elasticsearch와의 호환을 믿을 수 있어 더 편하게 설정할 수 있었다.   

추가적으로 Fluentd는 XML like 문법이라 설정이 개발자들에게 익숙하지 않았고, Logstash는 JRuby라는 자바스크립트 스러운 언어로 설정하여 훨씬 간단하고 익숙한 설정이 가능했다. (특히 if-else가 편했다.)

사실 Logstash가 메모리가 더 많이 사용되고, 성능상도 살짝 딸리다는 얘기도 들려왔다. 그래도 앞선 ELK 생태계, Elasticsearch + Kibana와의 호환성을 생각했을 때 Logstash + Filebeats 사용을 포기할 수 없는 옵션이었다. 

</br>

### 이번에 Elasticsearch로 로깅을 모니터링하면서 만난 꿀 기능

#### Index lifecycle policies
Index lifecycle policies가 너무 편하고 명확하게 기능하였다.   
Hot phase, Warm phase, Cold phase, Delete phase로 각 phase를 나눠 데이터의 생명을 관리했는데, 각각 접근 가능과 집계를 위해 사용되는 것에서부터 검색과 집계가 안되는 단순한 백업용 데이터로의 단계를 쉽게 설정할 수 있었다. 

![image](https://user-images.githubusercontent.com/46060746/197747038-63a24f85-1b6e-4e03-9cd3-5ffe2037a5e4.png)

(추가적으로 우리 팀은 아직 시험 단계라 ELK 스택을 POD으로 띄우고 데이터를 EFS에 올리고 있는데, POD이 비정상 종료되면서 Elasticsearch 설정 데이터가 유실될 수 있는 상황을 걱정하여 따로 Instance를 생성하여 띄우는 것이 좋을지 고민하고 있다.)

</br>

#### APM
처리한 요청의 Throughput, Time spant, Latency 등을 상세하게 모니터링 할 수 있다. 한 요청 안에서 스텝이나 외부 모듈 접근 등의 단위로 성능을 확인할 수 있었다.

![image](https://user-images.githubusercontent.com/46060746/197746803-8ec6d239-d71d-4933-a418-6f42db006d01.png)

