##  Github action gradle cache

### Gradle Caching

```
- name: Gradle Caching
  uses: actions/cache@v3
  with:
    path: |
      ~/.gradle/caches
      ~/.gradle/wrapper
    key: ${{ runner.os }}-gradle-${{ hashFiles('**/*.gradle*', '**/gradle-wrapper.properties') }}
```

path는 cache를 저장하는 경로인 것 같고,    
key는 캐시 키를 말하는 것 같다. {runner.os}가 바뀌면 gradle 파일이 같아도 같은 캐시 값이 아닐테니 의미가 없을 것이니 키 prefix로 os를 추가하고,    
grale 파일과 gradlew properties 파일들을 해시하여 postfix 하는 것으로 그레들 파일이 같음을 보장한다.   

만약 ruuner.os가 다르거나 gradle 파일 중 하나라도 수정이 된다면 캐시 미스를 발생시키도록 하는 것이다.   

### Performance 

#### cache load

cache load 에서 약간의 시간이 걸린다. 당연하겠지만. 근데 그 시간이 빌드 시간보다 길면 의미가 없을 것 같다.    
나는 HA / HPA를 위해 k8s cluster에 runner를 따로 띄워서 runner를 실행해서 느린건지 더 지켜봐야겠다.   
<img width="1097" alt="image" src="https://github.com/ecsimsw/daily-note-public/assets/46060746/0cc8c0b9-1cc3-4419-9ad3-fdd5c06d4080">

#### build and push 

적용한 프로젝트가 여러 모듈을 갖고 있고 아래 테스트 결과는 container build 이후 push 까지 한 결과이다.    
가장 느린 모듈이 평균 23초에서 7초로 줄었다.    
다른 모듈까지 더 더하면 그 캐시의 효과는 훨씬 줄 것이다.       

모듈 중에 가장 짧은 빌드 시간인 모듈은 5초인데 만약 모듈이 하나고 빌드 시간이 이렇게 짧았으면 캐시가 더 방해됬을 거라고 생각한다.

before : max module -> avg 23s    
<img width="435" alt="image" src="https://github.com/ecsimsw/daily-note-public/assets/46060746/05183da2-39ad-4d09-911b-429e45e8f9ae">

after : max module -> avg 7s    
<img width="437" alt="image" src="https://github.com/ecsimsw/daily-note-public/assets/46060746/ab5f2827-9c08-4177-a185-407d458de7e6">




