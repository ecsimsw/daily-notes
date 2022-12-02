# How to create user, role on K8S

on premise k8s에 user와 role를 정의하여 팀간, 사원간 접근 권한을 제한해야 했다.

</br>

## 관리자 측

#### 1. ServiceAccount 를 생성한다.

```
apiVersion: v1
kind: ServiceAccount
metadata:
  name: <USER_NAME>
  namespace: <USING_NAMESPACE>
```


#### 2. 사용자 토큰 확인

ServiceAccount 생성 시 '<USER_NAME>-token-<RANDOM_STRING>' 꼴의 secret name를 얻을 수 있다. 

```
kubectl get secret
```
   
다음의 명령어로 해당 secret의 토큰을 확인한다.

```
kubectl get secret user1-token-2s4h8 -o jsonpath="{.data.token}" | base64 -D
```


e.g - token)
![image](https://user-images.githubusercontent.com/46060746/205318478-9d3784cd-6bad-47bb-bf6f-befd4d8d3a08.png)


#### 3. Role 또는 Cluster Role를 생성한다.

Role은 namespace 단위로, ClusterRole은 cluster 단위로 권한의 범위를 갖는다.

```
apiVersion: rbac.authorization.k8s.io/v1
kind: <Role 또는 ClusterRole>
metadata:
  namespace: <NAME_SPACE>
  name: <NAME>
rules:
- apiGroups: ["pods, services, jobs"] 
  resources: ["deployments", "replicasets", "pods"]
  verbs: ["get", "list", "watch", "create", "update", "patch", "delete"]
```


#### 4. 생성한 ServiceAccount와 Role(또는 ClusterRole) 를 Binding 한다.

```
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: <ROLE_BINDING_NAME>
  namespace: <USING_NAMESPACE>
subjects:
- kind: User
  name: <USER_NAME>
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role 
  name: <ROLE_NAME>
  apiGroup: rbac.authorization.k8s.io
```

```
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: <ROLE_BINDING_NAME>
subjects:
- kind: ServiceAccount
  name: <USER_NAME>
  namespace: <NAMESPACE_OF_SA>
roleRef:
  kind: ClusterRole
  name: <CLUSTER_ROLE_NAME>
  apiGroup: rbac.authorization.k8s.io
```

</br>

## 사용자 측

#### 1. 관리자에게서 받은 토큰으로 kubeconfig에 credential 를 등록한다.

```
kubectl config set-credentials user1 --token=<TOKEN>
```


#### 2. 사용하려는 Cluster와 인증/인가 받을 User 정보로 Context를 등록한다.

```
kubectl config set-context user1 --cluster=<CLUSTER_NAME> --user=<USER_NAME>
```


#### +. 반대로 kubeconfig에서 정보를 제거해야 한다면 다음의 명령어를 사용할 수 있다.

```
kubectl config delete-cluster <CLUSTER_NAME>
kubectl config unset users.<USER_NAME>  
kubectl config delete-context <CONTEXT_NAME>
```

#### 3. Context로 전환한다.

```
kubectl config use-context <CONTEXT_NAME>
```

</br>

## Tip

#### 1. 기본으로 제공되는 미리 정해진 Role이 있다.

![image](https://user-images.githubusercontent.com/46060746/205319739-04c7a95c-5764-4947-b11b-0c24f1b1c6ba.png)


#### 2. Role를 정의할 때 resourceNames로 사용할 resource를 이름에 따라 특정 할 수 있다.
```
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: default
  name: configmap-updater
rules:
- apiGroups: [""]
  resources: ["configmaps"]
  resourceNames: ["my-configmap"]
  verbs: ["update", "get"]
```
