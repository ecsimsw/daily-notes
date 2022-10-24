## Elastic search 아는 척하기

[Elastic 가이드 북](https://esbook.kimjmin.net/)    
[클러스터 성능 모니터링과 최적화하기](https://ncucu.me/188) 


### 기본 구조
Cluster    
Node  
Index (database)  

![image](https://user-images.githubusercontent.com/46060746/197483295-88c3c3d9-1bb1-4a97-95b3-a9ae6d121df6.png)


### Node의 종류
Master node : index, node를 관리 / 후보 지정 가능  
Data node : 데이터 저장 노드  
Ingest node : Logstash, Fluentd, Beats 등의 데이터 저장 전 전처리  
Coordinate node : aggregation 처리 (Data node가 search, indexing에 집중 할 수 있도록)  

### Split brain
클러스터끼리의 통신이 단절 되었을 때, 두 클러스터가 모두 살아있으면 데이터 정합성이 발생할 것이다. 그래서 Elastic search는 내부적으로 다수결 투표를 해서 이런 상황에서 사용을 중단할 클러스터를 결정하는데 이를 Master node의 개수로 한다.
만약 master node가 짝수개라면 클러스터로 나뉘고 다수결 투표를 했을 때 동점이 나오는 상황이 발생한다. 이런 상황을 막기 위해 master node를 홀수개로 하여 클러스터간 통신 단절 상황에서 한쪽을 명확하게 중단할 수 있도록 한다.
![image](https://user-images.githubusercontent.com/46060746/197481930-49dccf5a-6923-4da2-b81f-241d2f1e302f.png)

 
### Node status
GREEN : 모든 인덱스의 샤드가 정상적인 상태  
YELLOW : 일부 인덱스의 샤드가 정상적이지 않은 상태. 읽기/쓰기는 가능하나 복원에 문제가 생길 수 있다.   
RED : 일부 인덱스의 프라이머리 또는 샤드가 정상적이지 않은 상태. 읽기/쓰기 불가능, 복원에 문제가 생길 수 있다.   
 
 </br>
 
`GET /_cat/health?v`

```
1epoch      timestamp cluster     status node.total node.data shards pri relo init unassign pending_tasks max_task_wait_time active_shards_percent
21666593045 06:30:45  k8s-cluster yellow          1         1     36  36    0    0       17             0                  -                 67.9%
```

</br>
   
`GET /_cat/indices?v`

```
health status index                                     uuid                   pri rep docs.count docs.deleted store.size pri.store.size
yellow open   pivo-new-2022.10.23                       QZ-pc-CiTLqm44yenupO2g   1   1     195020            0     44.6mb         44.6mb
yellow open   apm-7.3.2-transaction-000001              C0U8DGI4TDCR5UqkEngg5Q   1   1     106369            0       70mb           70mb
yellow open   pivo-new-2022.10.24                       z5PozZokTNiRhl8k2jXolQ   1   1      50993            0     14.5mb         14.5mb
yellow open   console-2022.10.22                        LMY6xPoeRyOT1TT5C2iYBA   1   1         19            0     76.8kb         76.8kb
yellow open   console-2022.10.24                        DMJbq3jCQQaFDMLh_fzIIw   1   1        335            0    300.9kb        300.9kb
yellow open   .lists-default-000001                     Ar8r4WYbRh-y0ks7vPHzDg   1   1          0            0       225b           225b
yellow open   console-2022.10.23                        QTlraXoAQFCrxaVUmkVjMQ   1   1        175            0    321.1kb        321.1kb
yellow open   .items-default-000001                     11EmI1GKQtWTED43lQ0JUg   1   1          0            0       225b           225b
yellow open   pivo-new-2022.10.22                       S0x5veDYS9qFG_Z1dlWbHw   1   1     281145            0     75.3mb         75.3mb
yellow open   access-2022.10.24                         VRQ0eck8Q7WFd5Hz4LjHSw   1   1       2889            0      2.1mb          2.1mb
yellow open   apm-7.3.2-metric-000001                   yvcHYqTGT6yFO409nJlF0Q   1   1      68445            0     28.1mb         28.1mb
green  open   metrics-endpoint.metadata_current_default Pq4mmQTiTSeOBwypzXVjRA   1   0          0            0       225b           225b
yellow open   apm-7.3.2-error-000001                    3njC1k4BRQirgJJt_qs_Jg   1   1          0            0       225b           225b
yellow open   apm-7.3.2-error-000002                    lHSCmJk5QCWeZM0htOuZpA   1   1       1034            0      1.1mb          1.1mb
yellow open   apm-7.3.2-span-000001                     Yf2jWiZQRKW3PMTNDo3UtQ   1   1        129            0    132.6kb        132.6kb
yellow open   apm-7.3.2-span-000002                     fSXSVKTYTXWsraeTpsvrkg   1   1         30            0    145.1kb        145.1kb
yellow open   apm-7.3.2-onboarding-2022.10.22           WRKEhew_Q_itTyjRg652nw   1   1          2            0     13.8kb         13.8kb
yellow open   apm-7.3.2-onboarding-2022.10.24           CZnbnk5AQaa9Hb_engxgTQ   1   1          1            0        7kb            7kb
```

</br>
 
### 데이터 저장 위치
elasticsearch.yaml/spec/template/spec/containers/volumeMounts

```
- name: elasticsearch-data
  mountPath: /usr/share/elasticsearch/data
- name: elasticsearch-config
  mountPath: /usr/share/elasticsearch/config/elasticsearch.yml
```
