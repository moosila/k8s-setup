# k8s setup in ubuntu (QEMU/KVM, kubeadm, containerd, runc, calico)

```shell
# spec
# host: Ubuntu 22.04.1 LTS
# guest vms: Ubuntu 22.04.1 LTS
# virtualizer: QEMU/KVM Virtual Machine Manager 4.0.0

# disable swap
sudo swapon --show
sudo free -h
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

# create containerd configuration file, config.toml
sudo mkdir /etc/containerd
sudo sh -c 'containerd config default > /etc/containerd/config.toml'

# configure the systemd cgroup driver with runc
sudo nano /etc/containerd/config.toml # update SystemdCgroup = true and save
# [plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc]
#   ...
#   [plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc.options]
#     SystemdCgroup = true
sudo systemctl restart containerd # restart containerd

# install kubeadm, kubelet and kubectl
sudo apt-get update
sudo apt-get install -y apt-transport-https ca-certificates curl # install prerequesites, https, ca-certs and curl
sudo curl -fsSLo /usr/share/keyrings/kubernetes-archive-keyring.gpg https://packages.cloud.google.com/apt/doc/apt-key.gpg # download google gpgs for the packages
echo "deb [signed-by=/usr/share/keyrings/kubernetes-archive-keyring.gpg] https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list # add the Kubernetes apt repository
sudo apt-get update # update apt package index
sudo apt-get install -y kubelet=1.23.10-00 kubectl=1.23.10-00 kubeadm=1.23.10-00 # install kubelet, kubeadm and kubectl. set =version when need to install specific version
sudo apt-mark hold kubelet kubeadm kubectl # ping the versions

# ???? haven't we configured in previous step? configure cgroup driver (default systemd if not provided)

# create the cluster
sudo kubeadm init --pod-network-cidr=10.0.0.0/16 # inspect the output for errors if any
# provide --cri-socket=unix:///var/run/containerd/containerd.sock if there are multiple container runtimes
# alter the command arguments and/or use --ignore-preflight-errors=... and rerun the command to ignore **known** errors. 
# got two errors [ERROR FileContent--proc-sys-net-bridge-bridge-nf-call-iptables]: /proc/sys/net/bridge/bridge-nf-call-iptables does not exist and [ERROR FileContent--proc-sys-net-ipv4-ip_forward]: /proc/sys/net/ipv4/ip_forward contents are not set to 1
# ignored all of them, by issueing --ignore-preflight-errors=all

# kubeadm init should return "Your Kubernetes control-plane has initialized successfully!"
# note down the kubeadm join node command printed in the console. e.g. kubeadm join 192.168.122.222:6443 --token rvbpq2.grcp2d0d5do5m14l \
	--discovery-token-ca-cert-hash sha256:ca0c5f3fa9d98b6d711f20388e1875d2ee69b4f3d404383617854cfd2c87ccda 

# set the kubeconfig
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
# root user can just export KUBECONFIG=/etc/kubernetes/admin.conf instead

# test the cluster (master node/control plain)
kubectl get nodes # this should return the master node with NotReady status

# deploy a pod network to the cluster
# install calico using manifest file
curl https://raw.githubusercontent.com/projectcalico/calico/v3.24.1/manifests/calico.yaml -O # download the calico manifest
kubectl apply -f calico.yaml # install calico

# test the cluster (master node/control plain)
kubectl get nodes # this should return the master node with Ready status
kubectl get pods -n kube-system # this should return all ready list of pods, calico, coredns, etcd, apiserver, kube-controler-manager, kube-proxy and kube-scheduler
kubectl run nginx --image=nginx:latest # this should return pod/nginx created. It might not get scheduled as there could be a taint in the master node

# prepare the worker node host, remove swap, disable swap, install container runtime, containerd, # install container runtime (low-level), runc, install CNI plugins, create containerd configuration file, config.toml, configure the systemd cgroup driver with runc, install kubeadm, kubelet and kubectl

# add worker node (runt the saved command in the kubeadm init step)
kubeadm join 192.168.122.222:6443 --token rvbpq2.grcp2d0d5do5m14l --discovery-token-ca-cert-hash sha256:ca0c5f3fa9d98b6d711f20388e1875d2ee69b4f3d404383617854cfd2c87ccda 
# alter the command arguments and/or use --ignore-preflight-errors=... and rerun the command to ignore **known** errors. 
# got two errors [ERROR FileContent--proc-sys-net-bridge-bridge-nf-call-iptables]: /proc/sys/net/bridge/bridge-nf-call-iptables does not exist and [ERROR FileContent--proc-sys-net-ipv4-ip_forward]: /proc/sys/net/ipv4/ip_forward contents are not set to 1
# ignored all of them, by issueing --ignore-preflight-errors=all
# kubeadm join command should return "This node has joined the cluster:..."

# Run 'kubectl get nodes' on the control-plane to see this node join the cluster
```

# troubleshooting
https://www.thegeekdiary.com/troubleshooting-kubectl-error-the-connection-to-the-server-x-x-x-x6443-was-refused-did-you-specify-the-right-host-or-port/
