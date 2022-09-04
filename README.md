# k8s setup in ubuntu

```shell

# disable swap
sudo swapon --show
free -h
sudo swapoff -a
sudo nano /etc/fstab #delete line /swap.img       none    swap    sw      0       0
sudo swapon --show # this should not have any output

# install container runtime, containerd
curl -L https://github.com/containerd/containerd/releases/download/v1.6.8/containerd-1.6.8-linux-amd64.tar.gz -O
curl -L -O https://github.com/containerd/containerd/releases/download/v1.6.8/containerd-1.6.8-linux-amd64.tar.gz.sha256sum
sha256sum -c containerd-1.6.8-linux-amd64.tar.gz.sha256sum # this should return OK
sudo tar Cxzvf /usr/local  containerd-1.6.8-linux-amd64.tar.gz
sudo curl -L https://raw.githubusercontent.com/containerd/containerd/main/containerd.service -o /usr/local/lib/systemd/system/containerd.service --create-dirs # dowload systemd unit file
sudo systemctl daemon-reload
sudo systemctl enable --now containerd

# install container runtime (low-level), runc
curl -L https://github.com/opencontainers/runc/releases/download/v1.1.4/runc.amd64 -O
curl -L https://github.com/opencontainers/runc/releases/download/v1.1.4/runc.sha256sum -O
sha256sum -c runc.sha256sum  # it should return runc.amd64: OK
sudo install -m 755 runc.amd64 /usr/local/sbin/runc

# install CNI plugins
curl -L https://github.com/containernetworking/plugins/releases/download/v1.1.1/cni-plugins-linux-amd64-v1.1.1.tgz -O
curl -L https://github.com/containernetworking/plugins/releases/download/v1.1.1/cni-plugins-linux-amd64-v1.1.1.tgz.sha256 -O
sha256sum -c cni-plugins-linux-amd64-v1.1.1.tgz.sha256 # this should return cni-plugins-linux-amd64-v1.1.1.tgz: OK
sudo mkdir -p /opt/cni/bin
sudo tar Cxzvf /opt/cni/bin cni-plugins-linux-amd64-v1.1.1.tgz

# install kubeadm, kubelet and kubectl
sudo apt-get update
sudo apt-get install -y apt-transport-https ca-certificates curl # install prerequesites, https, ca-certs and curl
sudo curl -fsSLo /usr/share/keyrings/kubernetes-archive-keyring.gpg https://packages.cloud.google.com/apt/doc/apt-key.gpg # download google gpgs for the packages
echo "deb [signed-by=/usr/share/keyrings/kubernetes-archive-keyring.gpg] https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list # add the Kubernetes apt repository
sudo apt-get update # update apt package index
sudo apt-get install -y kubelet kubeadm kubectl # install kubelet, kubeadm and kubectl
sudo apt-mark hold kubelet kubeadm kubectl # ping the versions

# configure cgroup driver (default systemd if not provided)

# create the cluster
sudo apt-get update
sudo apt-get upgrade # to get the latest version of kubeadm
sudo kubeadm init --pod-network-cidr=10.0.0.0/16 --cri-socket=unix:///var/run/containerd/containerd.sock # use --ignore-preflight-errors=all to ignore known errors. I had to ignore [ERROR FileContent--proc-sys-net-bridge-bridge-nf-call-iptables]: /proc/sys/net/bridge/bridge-nf-call-iptables does not exist and [ERROR FileContent--proc-sys-net-ipv4-ip_forward]: /proc/sys/net/ipv4/ip_forward contents are not set to 1





```
