# Setup the jumpbox

You can follow kelseyhightower's directions without any changes to the code in the part: [Set Up The Jumpbox](https://github.com/kelseyhightower/kubernetes-the-hard-way/blob/master/docs/02-jumpbox.md). I will put the shortened version of the guide below for convenience:

Log in to the jumpbox:

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
  https://github.com/kelseyhightower/kubernetes-the-hard-way.git
```

Change into the `kubernetes-the-hard-way directory`:

```
cd kubernetes-the-hard-way
```

This will be the working directory for the rest of the tutorial. If you ever get lost run the pwd command to verify you are in the right directory when running commands on the jumpbox:

```
pwd
```

```
/root/kubernetes-the-hard-way
```

### Download Binaries

From the `kubernetes-the-hard-way` directory, create a `downloads` directory:

```
mkdir downloads
```

Download the binaries listed in the `downloads.txt` file using the `wget` command:

```
wget -q --show-progress \
  --https-only \
  --timestamping \
  -P downloads \
  -i downloads.txt
```

After the download is complete, check your downloads:

```
ls -loh downloads
```

```
total 584M
-rw-r--r-- 1 root  41M May  9 13:35 cni-plugins-linux-arm64-v1.3.0.tgz
-rw-r--r-- 1 root  34M Oct 26 15:21 containerd-1.7.8-linux-arm64.tar.gz
-rw-r--r-- 1 root  22M Aug 14 00:19 crictl-v1.28.0-linux-arm.tar.gz
-rw-r--r-- 1 root  15M Jul 11 02:30 etcd-v3.4.27-linux-arm64.tar.gz
-rw-r--r-- 1 root 111M Oct 18 07:34 kube-apiserver
-rw-r--r-- 1 root 107M Oct 18 07:34 kube-controller-manager
-rw-r--r-- 1 root  51M Oct 18 07:34 kube-proxy
-rw-r--r-- 1 root  52M Oct 18 07:34 kube-scheduler
-rw-r--r-- 1 root  46M Oct 18 07:34 kubectl
-rw-r--r-- 1 root 101M Oct 18 07:34 kubelet
-rw-r--r-- 1 root 9.6M Aug 10 18:57 runc.arm64
```

### Install kubectl

Use the chmod command to make the kubectl binary executable and move it to the /usr/local/bin/ directory:

```
{
  chmod +x downloads/kubectl
  cp downloads/kubectl /usr/local/bin/
}
```

```
kubectl version --client
```

```
Client Version: v1.28.3
Kustomize Version: v5.0.4-0.20230601165947-6ce0bf390ce3
```
