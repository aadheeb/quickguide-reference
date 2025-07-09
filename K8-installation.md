---
title: Deploy On-prem Kubernetes Cluster
parent: DevOps
---

# Deploy On-prem Kubernetes Cluster

## Setting up Docker Engine

Update the apt  package index and install packages to allow  apt to use a repository over HTTPS:

```bash
sudo apt-get update
sudo apt-get install \
    ca-certificates \
    curl \
    gnupg \
    lsb-release
```

Add Docker’s official GPG key:

```bash
sudo apt-get update
sudo apt-get install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc
```

# Add the repository to Apt sources:

```
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "${UBUNTU_CODENAME:-$VERSION_CODENAME}") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

sudo apt-get update
```

Install Docker Engine, containerd, and Docker Compose.

```bash
sudo apt-get update
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-compose-plugin
```


## Setup Kubernetes binaries

```bash
sudo curl -fsSLo /etc/apt/keyrings/kubernetes-archive-keyring.gpg https://packages.cloud.google.com/apt/doc/apt-key.gpg
echo "deb [signed-by=/etc/apt/keyrings/kubernetes-archive-keyring.gpg] https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list

sudo apt-get update
sudo apt-get install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl
```


## Setup cri-dockerd

```bash
git clone https://github.com/Mirantis/cri-dockerd.git

# Run these commands as root
###Install GO###
wget https://go.dev/dl/go1.23.5.linux-amd64.tar.gz
tar -C /usr/local -xzf go1.23.5.linux-amd64.tar.gz
export PATH=$PATH:/usr/local/go/bin
go version

cd cri-dockerd
mkdir bin
go build -o bin/cri-dockerd
mkdir -p /usr/local/bin
install -o root -g root -m 0755 bin/cri-dockerd /usr/local/bin/cri-dockerd
cp -a packaging/systemd/* /etc/systemd/system
sed -i -e 's,/usr/bin/cri-dockerd,/usr/local/bin/cri-dockerd,' /etc/systemd/system/cri-docker.service
systemctl daemon-reload
systemctl enable cri-docker.service
systemctl enable --now cri-docker.socket
```


## Deploy Cluster

### Initialize Control Plane

```bash
swapoff --all
sudo kubeadm init --pod-network-cidr=<pod-cidr> --apiserver-cert-extra-sans=<ip/host-to-connect-api> --cri-socket=unix:///var/run/cri-dockerd.sock
```

for Weave CNI use `10.32.0.0/12` as `--pod-network-cidr`

Follow on screen instructions to setup kubeconfig in master machine, store kube join command for later use to join nodes to the cluster

#### Deploy Weave CNI

```bash
kubectl apply -f https://github.com/weaveworks/weave/releases/download/v2.8.1/weave-daemonset-k8s.yaml
```

### Join Nodes

```bash
swapoff --all
kubeadm join <control-plane-> --token <token> \
    --discovery-token-ca-cert-hash <discovery-token-hash> --cri-socket=unix:///var/run/cri-dockerd.sock
```

