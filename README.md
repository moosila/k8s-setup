# k8s-setup

```shell

# disable swap
sudo swapon --show
free -h
sudo swapoff -a
sudo nano /etc/fstab #delete line /swap.img       none    swap    sw      0       0
sudo swapon --show # this should not have any output

```
