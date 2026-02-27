```
curl -s -o /dev/null -w "%{time_total}" http://localhost:11443/anything | awk '{printf "Total time: %.0f ms\n", $1*1000}'
```
