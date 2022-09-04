# k8s-setup

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


```
