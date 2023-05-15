## E20230413 jenkins auto re-run script.md

```
#!/bin/sh
ps_cnt=`ps -ef | grep -v grep | grep 'java -jar /cicd/jenkins/agent.jar -jnlpUrl https://www.MY_JENKINS.com/manage/computer/mobile%2Dmacpro/jenkins-agent.jnlp' | wc -l`
if [[ $ps_cnt -gt 0 ]];then
    echo "Already running" 
    ps -ef | grep -v grep | grep 'java -jar /cicd/jenkins/agent.jar -jnlpUrl https://www.MY_JENKINS.com/manage/computer/mobile%2Dmacpro/jenkins-agent.jnlp' | awk '{print $2}'
else
    open -a Docker
    nohup java -jar /cicd/jenkins/agent.jar -jnlpUrl https://www.MY_JENKINS.com/manage/computer/mobile%2Dmacpro/jenkins-agent.jnlp -secret {SECRET} -workDir "/cicd/jenkins" &
fi
```
