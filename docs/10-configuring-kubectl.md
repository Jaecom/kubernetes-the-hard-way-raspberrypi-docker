# Configuring kubectl for Remote Access

> [!NOTE]
> In this section, you can follow the steps without any changes in kelseyhightower's official guide: [10-configuring-kubectl.md](https://github.com/kelseyhightower/kubernetes-the-hard-way/blob/master/docs/10-configuring-kubectl.md). I added the shortened version of the code below for convenience.

## The Admin Kubernetes Configuration File

You should be able to ping `server.kubernetes.local` based on the /etc/hosts DNS entry from a previous lap:

```
curl -k --cacert ca.crt \
  https://server.kubernetes.local:6443/version
```

```
{
  "major": "1",
  "minor": "28",
  "gitVersion": "v1.28.3",
  "gitCommit": "a8a1abc25cad87333840cd7d54be2efaf31a3177",
  "gitTreeState": "clean",
  "buildDate": "2023-10-18T11:33:18Z",
  "goVersion": "go1.20.10",
  "compiler": "gc",
  "platform": "linux/arm64"
}
```

Generate a kubeconfig file suitable for authenticating as the `admin` user:

```
{
  kubectl config set-cluster kubernetes-the-hard-way \
    --certificate-authority=ca.crt \
    --embed-certs=true \
    --server=https://server.kubernetes.local:6443

  kubectl config set-credentials admin \
    --client-certificate=admin.crt \
    --client-key=admin.key

  kubectl config set-context kubernetes-the-hard-way \
    --cluster=kubernetes-the-hard-way \
    --user=admin

  kubectl config use-context kubernetes-the-hard-way
}
```

## Verification

Check the version of the remote Kubernetes cluster:

```
kubectl version
```

```
Client Version: v1.28.3
Kustomize Version: v5.0.4-0.20230601165947-6ce0bf390ce3
Server Version: v1.28.3
```

List the nodes in the remote Kubernetes cluster:

```
kubectl get nodes
```

```
NAME     STATUS   ROLES    AGE     VERSION
node-0   Ready    <none>   5m38s   v1.28.3
node-1   Ready    <none>   3m32s   v1.28.3
```

Next: [Configuring Pod Networks](https://github.com/Jaecom/kubernetes-the-hard-way-raspberrypi-docker/blob/main/docs/11-pod-network-routes.md)
