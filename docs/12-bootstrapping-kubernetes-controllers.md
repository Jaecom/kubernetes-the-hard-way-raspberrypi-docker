# Bootstrapping the Kubernetes Control Plane

## Prerequisites

In your `jumpbox` instance, copy Kubernetes binaries and systemd unit files to the `server` instance:

```
scp \
  downloads-amd/kube-apiserver \
  downloads-amd/kube-controller-manager \
  downloads-amd/kube-scheduler \
  downloads-amd/kubectl \
  units/kube-apiserver.service \
  units/kube-controller-manager.service \
  units/kube-scheduler.service \
  configs/kube-scheduler.yaml \
  configs/kube-apiserver-to-kubelet.yaml \
  root@server:~/
```

<details>
<summary><code>arm</code></summary>

```
scp \
  downloads-arm/kube-apiserver \
  downloads-arm/kube-controller-manager \
  downloads-arm/kube-scheduler \
  downloads-arm/kubectl \
  units/kube-apiserver.service \
  units/kube-controller-manager.service \
  units/kube-scheduler.service \
  configs/kube-scheduler.yaml \
  configs/kube-apiserver-to-kubelet.yaml \
  root@server:~/
```

</details>

<br/>

The commands in this lab must be run on the controller instance: server. Login to the controller instance using the ssh command. Example:

```
ssh root@server
```

## Provision the Kubernetes Control Plane

### Install kubernetes controller binaries

```
mkdir -p /etc/kubernetes/config

{
  chmod +x kube-apiserver \
    kube-controller-manager \
    kube-scheduler kubectl

  mv kube-apiserver \
    kube-controller-manager \
    kube-scheduler kubectl \
    /usr/local/bin/
}
```

### Configure the following: kube-apiserver, kube-controller-manager, kube-scheduler:

```

{
  mkdir -p /var/lib/kubernetes/

  mv ca.crt ca.key \
    kube-api-server.key kube-api-server.crt \
    service-accounts.key service-accounts.crt \
    encryption-config.yaml \
    /var/lib/kubernetes/
}

mv kube-apiserver.service \
  /etc/systemd/system/kube-apiserver.service

mv kube-controller-manager.kubeconfig /var/lib/kubernetes/

mv kube-controller-manager.service /etc/systemd/system/

mv kube-scheduler.kubeconfig /var/lib/kubernetes/

mv kube-scheduler.yaml /etc/kubernetes/config/

mv kube-scheduler.service /etc/systemd/system/
```

### Start the Controller Services

```
{
  systemctl daemon-reload

  systemctl enable kube-apiserver \
    kube-controller-manager kube-scheduler

  systemctl start kube-apiserver \
    kube-controller-manager kube-scheduler
}
```

Verification:

```
kubectl cluster-info \
  --kubeconfig admin.kubeconfig
```

```
Kubernetes control plane is running at https://127.0.0.1:6443
```

Check the status for any errors:

```
systemctl status kube-apiserver kube-controller-manager kube-scheduler
```

You should see some errors related to cluter role permissions. We will fix that in the next section.

## RBAC for Kubelet Authorization

Create the `system:kube-apiserver-to-kubelet` ClusterRole with permissions to access the Kubelet API and perform most common tasks associated with managing pods:

```
kubectl apply -f kube-apiserver-to-kubelet.yaml \
  --kubeconfig admin.kubeconfig
```

Restart the controller services to apply the cluster role permission

```
  systemctl daemon-reload

  systemctl restart kube-apiserver \
    kube-controller-manager kube-scheduler
```

Check the staus of `kube-apiserver` for any errors:

```
systemctl status kube-apiserver kube-controller-manager kube-scheduler
```

### Verification

In your `jumpbox` machine, run the following commands to test if the kubernetes control plane is up and reachable:

```
curl -k --cacert ca.crt https://server.kubernetes.local:6443/version
```

```
{
  "major": "1",
  "minor": "32",
  "gitVersion": "v1.32.0",
  "gitCommit": "70d3cc986aa8221cd1dfb1121852688902d3bf53",
  "gitTreeState": "clean",
  "buildDate": "2024-12-11T17:59:15Z",
  "goVersion": "go1.23.3",
  "compiler": "gc",
  "platform": "linux/amd64"
}
```

Next: [13 - Setting Up Worker Nodes](https://github.com/Jaecom/kubernetes-the-hard-way-raspberrypi-docker/blob/main/docs/13-bootstrapping-kubernetes-workers.md)
