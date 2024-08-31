# Bootstrapping the etcd Cluster

## Prerequisites

Copy `etcd` binaries and systemd unit files to the `server` instance:

```
scp \
  downloads/etcd-v3.4.27-linux-arm64.tar.gz \
  units/etcd.service \
  root@server:~/
```

The commands in this lab must be run on the server machine. Login to the server machine using the ssh command. Example:

```
ssh root@server
```

## Bootstrapping an etcd Cluster

### Install etcd binaries, configure etcd server, and create the systemd unit file:

```
{
  tar -xvf etcd-v3.4.27-linux-arm64.tar.gz
  mv etcd-v3.4.27-linux-arm64/etcd* /usr/local/bin/
}

{
  mkdir -p /etc/etcd /var/lib/etcd
  chmod 700 /var/lib/etcd
  cp ca.crt kube-api-server.key kube-api-server.crt \
    /etc/etcd/
}

mv etcd.service /etc/systemd/system/
```

### Start the etcd Server

```
{
  systemctl daemon-reload
  systemctl enable etcd
  systemctl start etcd
}
```

## Verification

List the etcd cluster members:

```
etcdctl member list
```

```
6702b0a34e2cfd39, started, controller, http://127.0.0.1:2380, http://127.0.0.1:2379, false
```

View the status of etcd server. Look out for any errors and warnings:

```
systemctl status etcd
```

Next: [Setting Up Kubernetes Controllers](https://github.com/Jaecom/kubernetes-the-hard-way-raspberrypi-docker/blob/main/docs/08-bootstrapping-kubernetes-controllers.md)