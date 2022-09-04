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


```
