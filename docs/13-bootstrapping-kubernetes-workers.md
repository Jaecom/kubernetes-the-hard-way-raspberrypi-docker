# Bootstrapping the Kubernetes Worker Nodes

## Prerequisites

From your `jumpbox` container, copy Kubernetes binaries and systemd unit files to each worker instance:

```
for host in node-0 node-1; do
  SUBNET=$(grep $host machines.txt | cut -d " " -f 4)
  sed "s|SUBNET|$SUBNET|g" \
    configs/10-bridge.conf > 10-bridge.conf

  sed "s|SUBNET|$SUBNET|g" \
    configs/kubelet-config.yaml > kubelet-config.yaml

  scp 10-bridge.conf kubelet-config.yaml \
  root@$host:~/
done
```

Copy the downloaded files to `node-0` and `node-1` instances. Please use the command below. Do not use the command in the official guide, as you will encounter errors due to the copying the template`kubelet-config.yaml` file which will replace the kubelet-config.yaml file you created beforehand.

```
# Node-0 amd packages
scp \
  downloads-amd/runc.amd64 \
  downloads-amd/crictl-v1.32.0-linux-amd64.tar.gz \
  downloads-amd/cni-plugins-linux-amd64-v1.6.0.tgz \
  downloads-amd/containerd-2.0.0-linux-amd64.tar.gz \
  downloads-amd/kubectl \
  downloads-amd/kubelet \
  downloads-amd/kube-proxy \
  configs/99-loopback.conf \
  configs/containerd-config.toml \
  configs/kube-proxy-config.yaml \
  units/containerd.service \
  units/kubelet.service \
  units/kube-proxy.service \
  root@node-0:~/

# Node-1 arm packages
scp \
  downloads-arm/runc.arm64 \
  downloads-arm/crictl-v1.32.0-linux-arm64.tar.gz \
  downloads-arm/cni-plugins-linux-arm64-v1.6.0.tgz \
  downloads-arm/containerd-2.0.0-linux-arm64.tar.gz \
  downloads-arm/kubectl \
  downloads-arm/kubelet \
  downloads-arm/kube-proxy \
  configs/99-loopback.conf \
  configs/containerd-config.toml \
  configs/kube-proxy-config.yaml \
  units/containerd.service \
  units/kubelet.service \
  units/kube-proxy.service \
  root@node-1:~/
```

The commands in this lab must be run on each worker instance: `node-0`, `node-1`. Login to the worker instance using the ssh command. Example:

```
ssh root@node-0
ssh root@node-1
```

## Provisioning a Kubernetes Worker Node

Install the OS dependencies:

```
{
  apt update
  apt -y install socat conntrack ipset kmod # Install kmod for overlay
}
```

### Disable Swap

Swap should already be disabled from your host raspberry pi machine. Check with the following:

```
swapon --show
```

You should see nothing outputed in the terminal. If something is outputted, you need to diable swap in your host raspberry pi using the [03-raspberry-pi-setup](https://github.com/Jaecom/kubernetes-the-hard-way-raspberrypi-docker/blob/main/docs/03-raspberry-pi-setup.md) guide and restart your docker containers.

### Create the installation directories

```
mkdir -p \
  /etc/cni/net.d \
  /opt/cni/bin \
  /var/lib/kubelet \
  /var/lib/kube-proxy \
  /var/lib/kubernetes \
  /var/run/kubernetes

{
  mkdir -p containerd
  tar -xvf crictl-v1.32.0-linux-amd64.tar.gz
  tar -xvf containerd-2.0.0-linux-amd64.tar.gz -C containerd
  tar -xvf cni-plugins-linux-amd64-v1.6.0.tgz -C /opt/cni/bin/
  mv runc.amd64 runc
  chmod +x crictl kubectl kube-proxy kubelet runc
  mv crictl kubectl kube-proxy kubelet runc /usr/local/bin/
  mv containerd/bin/* /bin/
}
```

<details>
<summary><code>arm</code></summary>

```
mkdir -p \
  /etc/cni/net.d \
  /opt/cni/bin \
  /var/lib/kubelet \
  /var/lib/kube-proxy \
  /var/lib/kubernetes \
  /var/run/kubernetes

{
  mkdir -p containerd
  tar -xvf crictl-v1.32.0-linux-arm64.tar.gz
  tar -xvf containerd-2.0.0-linux-arm64.tar.gz -C containerd
  tar -xvf cni-plugins-linux-arm64-v1.6.0.tgz -C /opt/cni/bin/
  mv runc.arm64 runc
  chmod +x crictl kubectl kube-proxy kubelet runc
  mv crictl kubectl kube-proxy kubelet runc /usr/local/bin/
  mv containerd/bin/* /bin/
}
```

</details>

### Congfigure Services

```
mv 10-bridge.conf 99-loopback.conf /etc/cni/net.d/

# Configuring containerd
{
  mkdir -p /etc/containerd/
  mv containerd-config.toml /etc/containerd/config.toml
  mv containerd.service /etc/systemd/system/
}

# Configure the Kubelet
{
  mv kubelet-config.yaml /var/lib/kubelet/
  mv kubelet.service /etc/systemd/system/
}

# Configure the Kubernetes Proxy
{
  mv kube-proxy-config.yaml /var/lib/kube-proxy/
  mv kube-proxy.service /etc/systemd/system/
}
```

### Start the Worker Services

```
{
  systemctl daemon-reload
  systemctl enable containerd kubelet kube-proxy
  systemctl start containerd kubelet kube-proxy
}
```

Check the status of `containerd`, `kubelet`, and `kube-proxy` for any errors:

```
  systemctl status containerd kubelet kube-proxy
```

You may need to wait for a few seconds for the kubelet service to be fully operational.

Also check kernal logs for any errors regarding the file system:

```
  dmesg
```

## Verification

From your jumpbox machine, check if the nodes are correctly set up and ready:

```
ssh root@server \
  "kubectl get nodes \
  --kubeconfig admin.kubeconfig"
```

```
NAME     STATUS   ROLES    AGE    VERSION
node-0   Ready    <none>   1m     v1.28.3
node-1   Ready    <none>   10s    v1.28.3
```

Next: [14 - Setting Up Kubectl](https://github.com/Jaecom/kubernetes-the-hard-way-raspberrypi-docker/blob/main/docs/14-configuring-kubectl.md)
