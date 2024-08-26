# Provisioning Compute Resources

### Configuring ca.conf

First, in your raspberry pi machine, get the ip addresses of each container in docker:

```
sudo docker inspect -f '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' jumpbox server node-0 node-1
```

You should see something like this (in order):

```
172.17.0.2 #jumpbox
172.17.0.3 #server
172.17.0.4 #node-0
172.17.0.5 #node-1
```

Ssh into your jumpbox and edit the ca.conf file:

```
ssh root@localhost -p 2222
cd kubernetes-the-hard-way
vim ca.conf
```

Revised ca.conf based on ip addresses of docker containers. The revised sections are listed below with a #. You can find the full file here: [ca.conf]("https://github/Jaecom/kubernetes-the-hard-way-raspberrypi-docker).

```
...

[node-0_req_extensions]
basicConstraints     = CA:FALSE
extendedKeyUsage     = clientAuth, serverAuth
keyUsage             = critical, digitalSignature, keyEncipherment
nsCertType           = client
nsComment            = "Node-0 Certificate"
subjectAltName       = DNS:node-0, IP:172.17.0.4  # IP for node-0
subjectKeyIdentifier = hash

...

[node-1_req_extensions]
basicConstraints     = CA:FALSE
extendedKeyUsage     = clientAuth, serverAuth
keyUsage             = critical, digitalSignature, keyEncipherment
nsCertType           = client
nsComment            = "Node-1 Certificate"
subjectAltName       = DNS:node-1, IP:172.17.0.5  # IP for node-1
subjectKeyIdentifier = hash

...

[kube-proxy_alt_names]
IP.0  = 172.17.0.4  # IP for node-0
IP.1  = 172.17.0.5  # IP for node-1

...

[kube-controller-manager_req_extensions]
basicConstraints     = CA:FALSE
extendedKeyUsage     = clientAuth, serverAuth
keyUsage             = critical, digitalSignature, keyEncipherment
nsCertType           = client
nsComment            = "Kube Controller Manager Certificate"
subjectAltName       = DNS:kube-controller-manager, IP:172.17.0.3  # IP of the server container
subjectKeyIdentifier = hash

...

[kube-scheduler_req_extensions]
basicConstraints     = CA:FALSE
extendedKeyUsage     = clientAuth, serverAuth
keyUsage             = critical, digitalSignature, keyEncipherment
nsCertType           = client
nsComment            = "Kube Scheduler Certificate"
subjectAltName       = DNS:kube-scheduler, IP:172.17.0.3  # IP of the server container
subjectKeyIdentifier = hash

...

[kube-api-server_alt_names]
IP.0  = 172.17.0.3  # IP of the server container
IP.1  = 10.32.0.1
IP.2  = 127.0.0.1
DNS.0 = kubernetes
DNS.1 = kubernetes.default
DNS.2 = kubernetes.default.svc
DNS.3 = kubernetes.default.svc.cluster
DNS.4 = kubernetes.svc.cluster.local
DNS.5 = server.kubernetes.local
DNS.6 = api-server.kubernetes.local

```

> [!NOTE]
> From now on you can follow the rest of kelseyhightower's official guide in [03-compute-resources](https://github.com/kelseyhightower/kubernetes-the-hard-way/blob/master/docs/03-compute-resources.md). I added the shortened version of the code below for convenience.

## Machines.txt

Create machines.txt. Replace the ip addresses in the beginning with the appropriate ip addresses of your docker containers.

```
172.17.0.3 server.kubernetes.local server
172.17.0.4 node-0.kubernetes.local node-0 10.200.0.0/24
172.17.0.5 node-1.kubernetes.local node-1 10.200.1.0/24
```

## Configuring SSH

Enabling root ssh access is already done via the base docker container `debian-bookworm-ssh` that you created earlier in this guide.

### Generate and Destribute SSH Keys

Generate new SSH Key:

```
ssh-keygen
```

````
Generating public/private rsa key pair.
Enter file in which to save the key (/root/.ssh/id_rsa):
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in /root/.ssh/id_rsa
Your public key has been saved in /root/.ssh/id_rsa.pub```

````

Copy the SSH public key to each machine:

```
while read IP FQDN HOST SUBNET; do
  ssh-copy-id root@${IP}
done < machines.txt
```

Check the public key access is working:

```
while read IP FQDN HOST SUBNET; do
  ssh -n root@${IP} uname -o -m
done < machines.txt
```

```
aarch64 GNU/Linux
aarch64 GNU/Linux
aarch64 GNU/Linux
```

### Configure hostnames

Set the hostnames on each machine listed in the machines.txt file:

```
# Read from machines.txt
while read IP FQDN HOST SUBNET; do
    echo "Processing $IP ($FQDN, $HOST, $SUBNET)"

    # Append the FQDN and hostname to /etc/hosts on the remote machine
    ssh -n root@${IP} "echo '127.0.1.1 ${FQDN} ${HOST}' >> /etc/hosts"

    # Remove duplicate lines in /etc/hosts on the remote machine
    ssh -n root@${IP} "awk '!seen[\$0]++' /etc/hosts > /tmp/hosts && mv /tmp/hosts /etc/hosts"

    # Write the hostname to /etc/hostname on the remote machine
    ssh -n root@${IP} "echo ${HOST} > /etc/hostname"

    # Set the current hostname using the hostname command on the remote machine
    ssh -n root@${IP} "hostname ${HOST}"
done < machines.txt
```

Verify the hostname is set on each machine:

```
while read IP FQDN HOST SUBNET; do
  ssh -n root@${IP} hostname --fqdn
done < machines.txt
```

```
server.kubernetes.local
node-0.kubernetes.local
node-1.kubernetes.local
```

## DNS

Create new hosts file and add header to identify the machines being added:

```
echo "" > hosts
echo "# Kubernetes The Hard Way" >> hosts
```

Generate a DNS entry for each machine in the machines.txt file and append it to the hosts file:

```
while read IP FQDN HOST SUBNET; do
    ENTRY="${IP} ${FQDN} ${HOST}"
    echo $ENTRY >> hosts
done < machines.txt
```

Review the DNS entries in the hosts file:

```
cat hosts
```

```
# Kubernetes The Hard Way
172.17.0.3 server.kubernetes.local server
172.17.0.4 node-0.kubernetes.local node-0
172.17.0.5 node-1.kubernetes.local node-1
```

## Adding DNS Entries To A Local Machine

Append the DNS entries from hosts to /etc/hosts:

```
cat hosts >> /etc/hosts

```

Verify that the /etc/hosts file has been updated:

```
cat /etc/hosts

```

```
127.0.0.1       localhost
127.0.1.1       jumpbox

# The following lines are desirable for IPv6 capable hosts
::1     localhost ip6-localhost ip6-loopback
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters



# Kubernetes The Hard Way
XXX.XXX.XXX.XXX server.kubernetes.local server
XXX.XXX.XXX.XXX node-0.kubernetes.local node-0
XXX.XXX.XXX.XXX node-1.kubernetes.local node-1
```

At this point you should be able to SSH to each machine listed in the machines.txt file using a hostname.

```
for host in server node-0 node-1
   do ssh root@${host} uname -o -m -n
done
```

```
server aarch64 GNU/Linux
node-0 aarch64 GNU/Linux
node-1 aarch64 GNU/Linux
```

## Adding DNS Entries To The Remote Machines

Copy the hosts file to each machine and append the contents to /etc/hosts:

```
while read IP FQDN HOST SUBNET; do
  scp hosts root@${HOST}:~/
  ssh -n \
    root@${HOST} "cat hosts >> /etc/hosts"
done < machines.txt
```

Next: [Generating Certificates](https://github.com/Jaecom/kubernetes-the-hard-way-raspberrypi-docker/blob/main/docs/04-certificate-authority.md)