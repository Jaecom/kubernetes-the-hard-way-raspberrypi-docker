# Setting up raspberry pi

We will first do some configurations on your raspberry pi to prevent errors from happening later in this guide.

## SSH into your raspberry pi

```
ssh username@raspberry-pi-ip-address -p 22
```

## Disable Swap

Disable swap in your raspberry pi.

```
sudo swapon --show
sudo swapoff -a
```

Check the pi's memory usage

```
free -h
```

You should see something like this:

```
               total        used        free      shared  buff/cache   available
Mem:           7.9Gi       1.5Gi       1.8Gi       141Mi       4.8Gi       6.4Gi
Swap:             0B          0B          0B
```

## Include memory in cgroup

Install the `cgroup-tools`

```
sudo apt install cgroup-tools
```

Edit the `cmdLine.txt` file and reboot:

```
sudo nano /boot/firmware/cmdline.txt

#append at the end of line with a space
cgroup_enable=memory cgroup_memory=1

sudo reboot
```

Check the cgroup of your pi:

```
lscgroup
```

You should see memeory included in the cgroup lists:

```
cpuset,cpu,io,memory,pids:/
cpuset,cpu,io,memory,pids:/sys-fs-fuse-connections.mount
cpuset,cpu,io,memory,pids:/sys-kernel-config.mount
cpuset,cpu,io,memory,pids:/sys-kernel-debug.mount
cpuset,cpu,io,memory,pids:/dev-mqueue.mount
...and on
```

## Docker

You must have docker installed in your raspberry pi. In case of a debian machine, use the offical docs: [Install Docker Engine of Debian](https://docs.docker.com/engine/install/debian/)

Next: [Running Docker Containers](https://github.com/Jaecom/kubernetes-the-hard-way-raspberrypi-docker/blob/main/docs/01-docker-container-setup.md)
