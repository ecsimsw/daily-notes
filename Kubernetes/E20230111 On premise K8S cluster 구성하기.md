# How to configure cluster on proxmox

## Commons

### Set static ip

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

```
netplan apply
```

#### Disable swap
```
sudo -i
swapoff -a
echo 0 > /proc/sys/vm/swappiness
sed -e '/swap/ s/^#*/#/' -i /etc/fstab
```

#### Install crio v1.23
```
sudo apt update -y && 
sudo apt upgrade -y &&

OS=xUbuntu_20.04 &&
CRIO_VERSION=1.23 &&

echo "deb https://download.opensuse.org/repositories/devel:/kubic:/libcontainers:/stable/$OS/ /"|sudo tee /etc/apt/sources.list.d/devel:kubic:libcontainers:stable.list &&
echo "deb http://download.opensuse.org/repositories/devel:/kubic:/libcontainers:/stable:/cri-o:/$CRIO_VERSION/$OS/ /"|sudo tee /etc/apt/sources.list.d/devel:kubic:libcontainers:stable:cri-o:$CRIO_VERSION.list &&

curl -L https://download.opensuse.org/repositories/devel:kubic:libcontainers:stable:cri-o:$CRIO_VERSION/$OS/Release.key | sudo apt-key add - &&
curl -L https://download.opensuse.org/repositories/devel:/kubic:/libcontainers:/stable/$OS/Release.key | sudo apt-key add - && 

sudo apt update -y &&
sudo apt install cri-o cri-o-runc -y &&

sudo systemctl enable crio.service &&
sudo systemctl start crio.service &&

systemctl status crio
```

#### Install Kubeadm, Kubelet, Kubectl
```
sudo apt-get update -y &&
sudo apt-get install -y apt-transport-https ca-certificates curl &&

sudo curl -fsSLo /usr/share/keyrings/kubernetes-archive-keyring.gpg https://packages.cloud.google.com/apt/doc/apt-key.gpg &&
echo "deb [signed-by=/usr/share/keyrings/kubernetes-archive-keyring.gpg] https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list &&

sudo apt-get update &&
sudo apt-get install -y kubelet kubeadm kubectl &&
sudo apt-mark hold kubelet kubeadm kubectl
```

#### Network config
```
modprobe br_netfilter &&
echo '1' > /proc/sys/net/ipv4/ip_forward
```

#### Declare node-ip on kubelet environment values
```
NODE_IP=192.168.52.10 
echo "KUBELET_EXTRA_ARGS=\"--node-ip=$NODE_IP\"" >> /etc/default/kubelet
```

Declare `NODE_IP` according to your VM settings. Without this task, the node ip registered to the cluster is set to the default NAT ip address(10.0.2.15).

## Master node

#### Declare `NODE_IP`, `POD_NETWORK_CIDR` according to your configuration.

```
NODE_IP=192.168.52.10
POD_NETWORK_CIDR=172.16.0.0/16
```

#### kubeadm init

```
sudo kubeadm init \
--pod-network-cidr=$POD_NETWORK_CIDR \
--apiserver-advertise-address=$NODE_IP \
--control-plane-endpoint=$NODE_IP
```

#### Set user env value

root user
```
export KUBECONFIG=/etc/kubernetes/admin.conf
```

regular user
```
mkdir -p $HOME/.kube &&
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config &&
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

#### Check list
```
# Check pod network cidr
kubectl cluster-info dump | grep -m 1 cluster-cidr

# Check master node ip
kubectl get nodes -o wide

# Get join command
kubeadm token create --print-join-command
```

#### Get calico manifest file
```
CALICO_VERSION=3.25 &&
curl https://docs.projectcalico.org/archive/v$CALICO_VERSION/manifests/calico.yaml -O
```

#### Enable CALICO_IPV4POOL_CIDR and change the value as your pod network cidr
```
- name: CALICO_IPV4POOL_CIDR
  value: "172.16.0.0/16"
```

#### Apply
```
kubectl apply -f calico.yaml
```

## Worker node

#### Get join command from master node
```
kubeadm token create --print-join-command
```

#### Kubeadm join
```
kubeadm join 192.168.52.10:6443 --token 6iilcp.2uh68yajgxq95atw --discovery-token-ca-cert-hash sha256:4d2e33fa31dfd4596d76bc570f8849052b163e62662021ad6f344bdb2a3544cf 
```

#### Run 'kubectl get nodes' on master

```
kubectl get nodes -o wide
```
