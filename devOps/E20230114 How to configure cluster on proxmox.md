# Configure cluster on proxmox

## 0. Set static ip

`vi /etc/netplan/00-installer-config.yaml`

```
# This is the network config written by 'subiquity'
network:
  ethernets:
    ens18:
      dhcp4: false
      addresses: [10.1.254.12/16]       <---- Enter statis ip to use
      gateway4: 10.1.0.1
      nameservers:
        addresses: [8.8.8.8,8.8.4.4]
  version: 2
```

`netplan apply`

다른 서비스에서 전혀 사용되지 않는 내부 IP로 설정한다.



## 1. Disable swap file

```
sudo -i
swapoff -a
echo 0 > /proc/sys/vm/swappiness
sed -e '/swap/ s/^#*/#/' -i /etc/fstab 
```


## 2. install crio (Ubuntu 20.04.5 LTS)

```
sudo apt update && sudo apt upgrade

OS=xUbuntu_20.04
CRIO_VERSION=1.23
echo "deb https://download.opensuse.org/repositories/devel:/kubic:/libcontainers:/stable/$OS/ /"|sudo tee /etc/apt/sources.list.d/devel:kubic:libcontainers:stable.list
echo "deb http://download.opensuse.org/repositories/devel:/kubic:/libcontainers:/stable:/cri-o:/$CRIO_VERSION/$OS/ /"|sudo tee /etc/apt/sources.list.d/devel:kubic:libcontainers:stable:cri-o:$CRIO_VERSION.list

curl -L https://download.opensuse.org/repositories/devel:kubic:libcontainers:stable:cri-o:$CRIO_VERSION/$OS/Release.key | sudo apt-key add -
curl -L https://download.opensuse.org/repositories/devel:/kubic:/libcontainers:/stable/$OS/Release.key | sudo apt-key add -

sudo apt update
sudo apt install cri-o cri-o-runc

apt show cri-o

sudo systemctl enable crio.service
sudo systemctl start crio.service

systemctl status crio
```

https://computingforgeeks.com/install-cri-o-container-runtime-on-ubuntu-linux/

## 3. Kubeadm 

```
sudo apt-get update
sudo apt-get install -y apt-transport-https ca-certificates curl

sudo curl -fsSLo /usr/share/keyrings/kubernetes-archive-keyring.gpg https://packages.cloud.google.com/apt/doc/apt-key.gpg

echo "deb [signed-by=/usr/share/keyrings/kubernetes-archive-keyring.gpg] https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list

sudo apt-get update
sudo apt-get install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl
```


### MASTER NODE
```
sudo kubeadm init --pod-network-cidr=192.168.0.0/16

mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```


### WORKER NODE 

```
kubeadm join 10.1.250.1:6443 --token r3pfhd.la9qthynmoetyg4m --discovery-token-ca-cert-hash sha256:1ed9b24ff8d49227027800f5d359450f1d2aa1d2c2c793da1eb571fe05057caf --cri-socket=unix:///var/run/crio/crio.sock --node-name k8s-4-worker-401

unset KUBECONFIG
export KUBECONFIG=/etc/kubernetes/admin.conf
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```


### if) [ERROR FileContent--proc-sys-net-bridge-bridge-nf-call-iptables]: /proc/sys/net/bridge/bridge-nf-call-iptables does not exist

`modprobe br_netfilter`



### if) [ERROR FileContent--proc-sys-net-ipv4-ip_forward]: /proc/sys/net/ipv4/ip_forward contents are not set to 1

`echo '1' > /proc/sys/net/ipv4/ip_forward`



### if) [ERROR FileAvailable--etc-kubernetes-pki-ca.crt]: /etc/kubernetes/pki/ca.crt already exists

`rm -rf /etc/kubernetes/pki/ca.crt`



## 4. Install Calico

https://projectcalico.docs.tigera.io/getting-started/kubernetes/quickstart 

#### Install

```
kubectl create -f https://raw.githubusercontent.com/projectcalico/calico/v3.24.5/manifests/tigera-operator.yaml
kubectl create -f https://raw.githubusercontent.com/projectcalico/calico/v3.24.5/manifests/custom-resources.yaml
```

#### before calico

```
root@k8s-4:~# kubectl get nodes
NAME               STATUS     ROLES           AGE     VERSION
k8s-4              Ready      control-plane   6m51s   v1.26.0
k8s-4-worker-401   NotReady   <none>          1s      v1.26.0
```

#### after calico

```
root@k8s-4:~# kubectl get nodes
NAME               STATUS   ROLES           AGE     VERSION
k8s-4              Ready    control-plane   7m52s   v1.26.0
k8s-4-worker-401   Ready    <none>          62s     v1.26.0
```


## 5. Verify Calico

`vi dnsutils.yaml`

```
apiVersion: v1
kind: Pod
metadata:
  name: dnsutils
  namespace: default
spec:
  containers:
  - name: dnsutils
    image: registry.k8s.io/e2e-test-images/jessie-dnsutils:1.3
    command:
      - sleep
      - "infinity"
    imagePullPolicy: IfNotPresent
  restartPolicy: Always
```


`kubectl exec -it dnsutils /bin/bash`

```
nslookup google.com
```


### if) nslookup can’t find server
```
kubeadm reset
```
