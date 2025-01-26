# Setup the jumpbox

Log in to the jumpbox. If you did not change the default password, the password id `admin`:

```
ssh root@localhost -p 2222
```

### Install Command Line Utilities

```
apt update
apt -y install wget curl vim openssl git
```

### Clone Github Repo

```
git clone --depth 1 \
  https://github.com/Jaecom/kubernetes-the-hard-way-raspberrypi-docker.git
```

Change into the `kubernetes-the-hard-way-raspberrypi-docker` directory:

```
cd kubernetes-the-hard-way-raspberrypi-docker
```

This will be the working directory for the rest of the tutorial. If you ever get lost run the pwd command to verify you are in the right directory when running commands on the jumpbox:

```
pwd
```

```
/root/kubernetes-the-hard-way-raspberrypi-docker
```

### Download Binaries

From the `kubernetes-the-hard-way-raspberrypi-docker` directory, create a `downloads` directory depending on the system architecture:

```
mkdir downloads-amd
mkdir downloads-arm
```

Download the binaries listed in the `downloads-arm.txt` or `downloads-amd.txt` file using the `wget` command:

```
# Download binaries for amd
wget -q --show-progress \
  --https-only \
  --timestamping \
  -P downloads-amd \
  -i downloads-amd.txt

# Download binaries for arm
wget -q --show-progress \
  --https-only \
  --timestamping \
  -P downloads-arm \
  -i downloads-arm.txt
```

After the download is complete, check your downloads:

```
ls -loh downloads-arm
ls -loh downloads-amd
```

```
total 536M
-rw-r--r-- 1 root 48M Oct 15 09:37 cni-plugins-linux-arm64-v1.6.0.tgz
-rw-r--r-- 1 root 32M Nov  5 19:37 containerd-2.0.0-linux-arm64.tar.gz
-rw-r--r-- 1 root 17M Dec  9 09:16 crictl-v1.32.0-linux-arm64.tar.gz
-rw-r--r-- 1 root 16M Sep 11 18:28 etcd-v3.4.34-linux-arm64.tar.gz
-rw-r--r-- 1 root 87M Dec 11 21:12 kube-apiserver
-rw-r--r-- 1 root 80M Dec 11 21:12 kube-controller-manager
-rw-r--r-- 1 root 63M Dec 11 21:12 kube-proxy
-rw-r--r-- 1 root 62M Dec 11 21:12 kube-scheduler
-rw-r--r-- 1 root 54M Dec 11 21:12 kubectl
-rw-r--r-- 1 root 72M Dec 11 21:12 kubelet
-rw-r--r-- 1 root 11M Nov  1 22:23 runc.arm64

total 557M
-rw-r--r-- 1 root 51M Oct 15 09:37 cni-plugins-linux-amd64-v1.6.0.tgz
-rw-r--r-- 1 root 36M Nov  5 19:37 containerd-2.0.0-linux-amd64.tar.gz
-rw-r--r-- 1 root 19M Dec  9 09:16 crictl-v1.32.0-linux-amd64.tar.gz
-rw-r--r-- 1 root 17M Sep 11 18:28 etcd-v3.4.34-linux-amd64.tar.gz
-rw-r--r-- 1 root 89M Dec 11 21:12 kube-apiserver
-rw-r--r-- 1 root 82M Dec 11 21:12 kube-controller-manager
-rw-r--r-- 1 root 64M Dec 11 21:12 kube-proxy
-rw-r--r-- 1 root 63M Dec 11 21:12 kube-scheduler
-rw-r--r-- 1 root 55M Dec 11 21:12 kubectl
-rw-r--r-- 1 root 74M Dec 11 21:12 kubelet
-rw-r--r-- 1 root 11M Nov  1 22:23 runc.amd64
```

### Install kubectl

Use the chmod command to make the kubectl binary executable and move it to the /usr/local/bin/ directory:

```
{
  chmod +x downloads-amd/kubectl
  cp downloads-amd/kubectl /usr/local/bin/
}
```

<details>
<summary><code>arm</code></summary>

```
{
  chmod +x downloads-arm/kubectl
  cp downloads-arm/kubectl /usr/local/bin/
}
```

</details>

<br/>

```
kubectl version --client
```

```
Client Version: v1.32.0
Kustomize Version: v5.5.0
```

Next: [07 - Provisioning Resources](https://github.com/Jaecom/kubernetes-the-hard-way-raspberrypi-docker/blob/main/docs/07-compute-resources-ssh-config.md)
