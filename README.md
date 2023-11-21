# Install Kubernetes Cluster
<pre>
  Server : 1 Server Master & 1 Server Worker
  CPU    : Min 2 core
  RAM    : Min 2 GB
  OS     : Ubuntu Server 22.04
</pre>

## Setup Hostname
Server Master
```
$ sudo hostnamectl set-hostname "kube-master"
```
Server Worker
```
$ sudo hostnamectl set-hostname "kube-worker"
```

## Install Docker
Install for Server Master and Worker
#### Installing Docker
```
$ sudo apt update
```
```
$ sudo apt install apt-transport-https ca-certificates curl software-properties-common
```
```
$ curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
```
```
$ echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```
```
$ sudo apt update
```
```
$ sudo apt install docker-ce
```
#### Run Docker Without Sudo
```
$ sudo usermod -aG docker ${USER}
```
```
$ su - ${USER}
```
```
$ groups
```
```
$ docker version
```

## Install Container Runtime
Install for Server Master and Worker
#### Disable Swap
```
$ sudo swapoff -a
```
Comment line swap on `/etc/fstab`
```
$ sudo nano /etc/fstab
```
![image](https://github.com/fauzigalih/kubernetes/assets/64176403/63739b06-f3d4-402c-ba50-0263e7f48807)
```
$ mount -a
```
```
$ free -h
```

#### Install Go
Install for Server Master and Worker<br>
Download Go specific version on `https://go.dev/dl/` and extract to `/usr/local`
```
$ wget https://go.dev/dl/go1.21.0.linux-amd64.tar.gz
```
```
$ sudo tar -C /usr/local -xzf go1.21.0.linux-amd64.tar.gz
```
```
$ export PATH=$PATH:/usr/local/go/bin
```
```
$ go version
```

#### Install Cri Dockerd
Install for Server Master and Worker
```
$ git clone https://github.com/Mirantis/cri-dockerd.git
```
```
$ cd cri-dockerd
```
```
$ sudo apt install make
```
```
$ make cri-dockerd
```

Run these commands as root
```
# mkdir -p /usr/local/bin
```
```
# install -o root -g root -m 0755 cri-dockerd /usr/local/bin/cri-dockerd
```
```
# install packaging/systemd/* /etc/systemd/system
```
```
# sed -i -e 's,/usr/bin/cri-dockerd,/usr/local/bin/cri-dockerd,' /etc/systemd/system/cri-docker.service
```
```
# systemctl daemon-reload
```
```
# systemctl enable --now cri-docker.socket
```


## Installing kubeadm, kubelet and kubectl 
Install for Server Master and Worker
```
$ sudo apt-get update
```
```
$ sudo apt-get install -y apt-transport-https ca-certificates curl
```
```
$ curl -fsSL https://dl.k8s.io/apt/doc/apt-key.gpg | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-archive-keyring.gpg
```
```
$ echo "deb [signed-by=/etc/apt/keyrings/kubernetes-archive-keyring.gpg] https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list
```
```
$ sudo apt-get update
```
```
$ sudo apt-get install -y kubelet kubeadm kubectl
```
```
$ sudo apt-mark hold kubelet kubeadm kubectl
```

## Setup Master Kubernetes with Pod Network by Calico
Setup for Server Master<br>
Docs: `https://docs.tigera.io/calico/latest/getting-started/kubernetes/hardway/standing-up-kubernetes#install-kubernetes`
```
$ sudo kubeadm init --pod-network-cidr=192.168.0.0/16 --cri-socket=unix:///var/run/cri-dockerd.sock
```
```
$ mkdir -p $HOME/.kube
```
```
$ sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
```
```
$ sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

## Run Pod Network by Calico 
Run for Server Master<br>
Docs: `https://docs.tigera.io/calico/latest/getting-started/kubernetes/self-managed-onprem/onpremises#how-to`
```
$ curl https://raw.githubusercontent.com/projectcalico/calico/v3.26.1/manifests/calico.yaml -O
```
```
$ kubectl apply -f calico.yaml
```
Check pod network running:
```
$ kubectl get pod -A
```
Check node status:
```
$ kubectl get node
```

## Join Worker to Master Kubernetes
Run in server master:
```
$ kubeadm token create --print-join-command
```
Example:
![image](https://github.com/fauzigalih/kubernetes/assets/64176403/a7033130-4e88-4328-bf15-c89f3e7b4031)

Copy token and paste on server worker, and add `--cri-socket`:
```
$ sudo kubeadm join 172.16.27.76:6443 --token vln7up.vyu97ik03jt0pb11 --discovery-token-ca-cert-hash sha256:aec5b7948d8ea2aea4df3089f303d0bfb015a4ac6779592c014140ea3dcbbea4 --cri-socket=unix:///var/run/cri-dockerd.sock
```
Check node master and worker on server master:
```
$ kubectl get node
```
![image](https://github.com/fauzigalih/kubernetes/assets/64176403/b454fd03-2df1-491f-8531-9813bff1d81d)


#### Yeayyy, Finish....

