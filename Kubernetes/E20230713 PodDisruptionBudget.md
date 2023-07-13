## PodDisruptionBudget

Node가 drain 되었을 때 최소 pod 수는 보장되어야 한다. minAvailable은 최소 남길 pod 수를 지정하고, maxUnavailable은 사용 불가 상태, 이전할 pod 수를 지정한다.

``` yaml
apiVersion: policy/v1
kind: PodDisruptionBudget
metadata:
  name: main-server-pdb
  namespace: mymarket
spec:
  minAvailable: 2
  selector:
    matchLabels:
      app: products-server
```
