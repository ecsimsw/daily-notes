## Github actions self hosted runner

Github actions self hosted runner로 github hosting에서 벗어났다.     
Organization 범위의 Actions -> Runner를 등록하면 포함된 Repository는 모두 해당 Runner를 사용할 수 있다.   
각 Runner가 하나의 프로세스로 여러 Job을 동시에 처리하고 싶다면 여러 Runner를 실행해야 한다.   

나는 두 빌드 노드에 각각 2개씩 Runner를 실행시켰다. (총 4개의 Job이 병렬 실행 가능)

## Github actions health check and run script 


```
#!/bin/sh

input_count=2
process_count=$(ps -ef | grep -v grep | grep 'githubActions/actions-runner.*bin/Runner.Listener run'| wc -l | tr -d " ")
difference=$((input_count-process_count))

if [[ $difference == 0 ]];then
  echo "all runner are healthy"
else
  for (( i=1; i<=input_count; i++ ))
    do
        echo "check runner : actions-runner${i}"
        ps_cnt=$(ps -ef | grep -v grep | grep "githubActions/actions-runner${i}/bin/Runner.Listener run" | wc -l | tr -d " ")
        if [[ $ps_cnt -gt 0 ]];then
          echo "actions-runner${i} is already running"
        else
          echo "start actions-runner${i}"
          nohup "/cicd/githubActions/actions-runner${i}/run.sh" &
        fi
    done
fi
```
위처럼 실행 스크립트를 작성했다.   

원하는 전체 프로세스 수를 결정하고 부족하다면 어떤 프로세스가 꺼졌는지를 확인하여 다시 실행시키는게 다이다.    
Github actions runner는 한 프로세스마다 각각의 폴더가 필요했다. 설정 정보를 따로 저장하나보다.

## Cron Job

```
SHELL=/bin/zsh
PATH=/usr/bin:/bin:/usr/sbin:/sbin:/usr/local/bin:/usr/local/bin:/opt/homebrew/bin:/opt/homebrew/sbin:/usr/local/bin:/usr/bin:/bin:/usr/sbin:/sbin

*/1 * * * * /cicd/githubActions/github-actions-macpro.sh >> /cicd/githubActions/github-actions-macpro.log 2>&1
59 23 * * * rm -rf /cicd/githubActions/github-actions-macpro.log 2>&1 && touch /cicd/githubActions/github-actions-macpro.log 
```

실행 스크립트를 crontab에 등록하여 1분에 한번씩 헬스 체크 -> 실행 시킨다.    
로그는 매일 밤 11시 59분에 삭제하도록.
