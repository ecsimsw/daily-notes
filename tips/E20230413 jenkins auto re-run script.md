## E20230413 jenkins auto re-run script.md

```
ps_cnt=`ps -ef | grep -v grep | grep 'java -jar agent.jar -jnlpUrl' | wc -l`
if [[ $ps_cnt -gt 0 ]];then
    echo "Already running" 
    ps -ef | grep -v grep | grep 'java -jar agent.jar {XXXX_URL}' | awk '{print $2}'
else
    nohup java -jar agent.jar -jnlpUrl {XXX_URL} -workDir "{WORK_DIR}" &
fi
```
