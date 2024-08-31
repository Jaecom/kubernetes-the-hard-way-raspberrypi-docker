# Provisioning a CA and Generating TLS Certificates

> [!NOTE]
> In this section, you can follow the steps without any changes in kelseyhightower's official guide: [04-certificate-authority](https://github.com/kelseyhightower/kubernetes-the-hard-way/blob/master/docs/04-certificate-authority.md). I added the shortened version of the code below for convenience.

## Certificate Authority:

Inside the `kubernetes-the-hard-way` directory, run generate the configuration file, certificate, and private key.

```
{
  openssl genrsa -out ca.key 4096
  openssl req -x509 -new -sha512 -noenc \
    -key ca.key -days 3653 \
    -config ca.conf \
    -out ca.crt
}
```

## Create Client and Server Certificates

Generate the certificates and private keys:

```
certs=(
  "admin" "node-0" "node-1"
  "kube-proxy" "kube-scheduler"
  "kube-controller-manager"
  "kube-api-server"
  "service-accounts"
)

for i in ${certs[*]}; do
  openssl genrsa -out "${i}.key" 4096

  openssl req -new -key "${i}.key" -sha256 \
    -config "ca.conf" -section ${i} \
    -out "${i}.csr"

  openssl x509 -req -days 3653 -in "${i}.csr" \
    -copy_extensions copyall \
    -sha256 -CA "ca.crt" \
    -CAkey "ca.key" \
    -CAcreateserial \
    -out "${i}.crt"
done
```

## Distribute the Certificates

Copy the appropriate certificates and private keys to the `node-0` and `node-1` machines:

```
for host in node-0 node-1; do
  ssh root@$host mkdir /var/lib/kubelet/

  scp ca.crt root@$host:/var/lib/kubelet/

  scp $host.crt \
    root@$host:/var/lib/kubelet/kubelet.crt

  scp $host.key \
    root@$host:/var/lib/kubelet/kubelet.key
done
```

Copy the appropriate certificates and private keys to the `server` machine:

```
scp \
  ca.key ca.crt \
  kube-api-server.key kube-api-server.crt \
  service-accounts.key service-accounts.crt \
  root@server:~/
```

Next: [Setting up Kubernetes Config](https://github.com/Jaecom/kubernetes-the-hard-way-raspberrypi-docker/blob/main/docs/05-kubernetes-configuration-files.md)
