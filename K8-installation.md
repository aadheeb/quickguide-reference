**Kubernetes Installation Commands**


- `sudo apt-get update`

- `sudo apt-get install -y apt-transport-https ca-certificates curl`

- `sudo curl -fsSLo /etc/apt/keyrings/kubernetes-archive-keyring.gpg https://packages.cloud.google.com/apt/doc/apt-key.gpg echo "deb [signed-by=/etc/apt/keyrings/kubernetes-archive-keyring.gpg] https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list`

- `sudo apt-get update`

- `sudo apt-get install -y kubelet kubeadm kubectl`

- `sudo apt-mark hold kubelet kubeadm kubectl`

**To build for a specific architecture, add ARCH= as an argument, where ARCH is a known build target for golang**

**Install Golang**

Clone this Repo
	`git clone https://github.com/Mirantis/cri-dockerd.git`

#Run these commands as root

- `wget https://storage.googleapis.com/golang/getgo/installer_linux`

- `chmod +x ./installer_linux`

- `./installer_linux`

- `source ~/.bash_profile`

#Then setup cri-dockerd

- `cd cri-dockerd`

- `mkdir bin`

- `go build -o bin/cri-dockerd`

- `mkdir -p /usr/local/bin`

- `install -o root -g root -m 0755 bin/cri-dockerd /usr/local/bin/cri-dockerd`

- `cp -a packaging/systemd/* /etc/systemd/system`

- `sed -i -e 's,/usr/bin/cri-dockerd,/usr/local/bin/cri-dockerd,' /etc/systemd/system/cri-docker.service`

- `systemctl daemon-reload`

- `systemctl enable cri-docker.service`

- `systemctl enable --now cri-docker.socket`


**Kubeadm join command**

- `swapoff --all`

 ```
- kubeadm join 192.168.1.140:6443 --token 3nz4mb.jastxq0hguruiwkp \
    --discovery-token-ca-cert-hash sha256:2fd4f54dd1e303e5f37e7572e2c8653747980c36dca14ba5d1eb6300f20cca08 --cri-socket=unix:///var/run/cri-dockerd.sock
    ```