## 0. Task

이슈 : Jenkins agent 가 외부 상황에 따라 간헐적으로 다운됨
처리 : Mac boot 시 또는 1분마다 한번씩 프로세스를 확인하여 없다면 agent 실행하는 것으로 수정

## 1. Jenkins agent run script  

### 1-1. Jenkins run script
```
#!/bin/sh
ps_cnt=`ps -ef | grep -v grep | grep '{JENKINS_DOMAIN_NAME}/jenkins-agent.jnlp' | wc -l`
if [[ $ps_cnt -gt 0 ]];then
    echo "Already running" 
    ps -ef | grep -v grep | grep '{JENKINS_DOMAIN_NAME}/jenkins-agent.jnlp' | awk '{print $2}'
else
    open -a Docker
    nohup java -jar {JENKINS_AGENT_PATH}/agent.jar -jnlpUrl {JENKINS_DOMAIN_NAME}/jenkins-agent.jnlp -secret {SECRET} -workDir "{WORK_DIRECTORY}" &
fi
```
### 1-2. Process
- check is running 
- if it’s already running
  - print PID of running Jenkins process
- if not 
  - run docker daemon 
  - run Jenkins agent

## 2. Execute Jenkins run script on boot

### 2-1 Make .command file

```
mv jenkins-agent-run.sh jenkins-agent-run.sh.command
```

### 2-2 Add command file as Login item

```
System Preferences -> Users & Groups -> Login items
```

## 3. Register Crontab

### 3-1 Edit crontab

edit cron job
```
crontab -e
```

look cron job list
```
crontab -l
```

### 3-2 Register health check script as cron job
```
SHELL=/bin/zsh

*/1 * * * * /bin/zsh {JENKINS_AGENT_RUN_SCRIPT_PATH}.sh >> {CRONTAB_LOG_PATH}.log 2>&1
59 23 * * * rm -rf {CRONTAB_LOG_PATH}.log 2>&1 && touch {CRONTAB_LOG_PATH}.log
```

- execute health check script every 1 min
- refresh cronjob log file at everyday PM 11:59
