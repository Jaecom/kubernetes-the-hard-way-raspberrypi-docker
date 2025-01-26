# Provisioning Compute Resources

### Configuring ca.conf

Ssh into your `jumpbox` and edit the ca.conf file:

```
ssh root@localhost -p 2222
```

```
cd kubernetes-the-hard-way-raspberrypi-docker
vim ca.conf
```

## Machines.txt

Create `machines.txt`. Replace the ip addresses in the beginning with the appropriate ip addresses of your docker containers.

```
vim machines.txt
```

```
10.0.0.1 server.kubernetes.local server 10.244.0.0/24
10.0.1.1 node-0.kubernetes.local node-0 10.244.1.0/24
10.0.2.1 node-1.kubernetes.local node-1 10.244.2.0/24
```

## Configuring SSH

Enabling root ssh access is already done via the base docker container `debian-bookworm-ssh` that you created earlier in this guide.

### Edit SSH Config

```
vim ~/.ssh/config
```

Edit the `SSH` config file. You can input a different `IP` and `port` if using a remote host.

```
Host server
  HostName 10.0.0.1 # IP Address of server
  User root
  Port 22

Host node-0
  HostName 10.0.1.1 # IP Address of node-0
  User root
  Port 6001

Host node-1
  HostName 10.0.2.1 # IP Address of node-1
  User root
  Port 6001
```

> [!Note]
> The reason why we use `port 6001` for the `node-0` and `node-1` containers is because we used `--network host` when running the node containers. Since the docker containers will use the host's network stack, `port 22` is already taken so we specified `ssh` to be configured in the `6001 port` in `debian-bookworm-ssh`.

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
  ssh-copy-id root@${HOST}
done < machines.txt
```

Check that SSH is working for the other containers:

```
while read IP FQDN HOST SUBNET; do
  ssh -n root@${HOST} uname -o -m
done < machines.txt
```

```
x86_64 GNU/Linux # EC2 A
x86_64 GNU/Linux # EC2 B
aarch64 GNU/Linux # Raspberry Pi
```

### Configure hostnames

Set the hostnames on each machine listed in the machines.txt file:

```
# Read from machines.txt
while read IP FQDN HOST SUBNET; do
    echo "Processing $IP ($FQDN, $HOST, $SUBNET)"

    # Append the FQDN and hostname to /etc/hosts on the remote machine
    ssh -n root@${HOST} "echo '127.0.1.1 ${FQDN} ${HOST}' >> /etc/hosts"

    # Write the hostname to /etc/hostname on the remote machine
    ssh -n root@${HOST} "echo ${HOST} > /etc/hostname"

    # Set the current hostname using the hostname command on the remote machine
    ssh -n root@${HOST} "hostname ${HOST}"
done < machines.txt
```

Verify the hostname is set on each machine:

```
while read IP FQDN HOST SUBNET; do
  ssh -n root@${HOST} hostname --fqdn
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
10.0.0.1 server.kubernetes.local server
10.0.1.1 node-0.kubernetes.local node-0
10.0.2.1 node-1.kubernetes.local node-1
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
127.0.0.1	localhost
::1	localhost ip6-localhost ip6-loopback
fe00::0	ip6-localnet
ff00::0	ip6-mcastprefix
ff02::1	ip6-allnodes
ff02::2	ip6-allrouters
172.17.0.2	333815efab69

# Kubernetes The Hard Way
10.0.0.1 server.kubernetes.local server
10.0.1.1 node-0.kubernetes.local node-0
10.0.2.1 node-1.kubernetes.local node-1
```

At this point you should be able to SSH to each machine listed in the machines.txt file using a hostname.

```
for host in server node-0 node-1
   do ssh root@${host} uname -o -m -n
done
```

```
server x86_64 GNU/Linux
node-0 x86_64 GNU/Linux
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

Next: [08 - Generating Certificates](https://github.com/Jaecom/kubernetes-the-hard-way-raspberrypi-docker/blob/main/docs/08-certificate-authority.md)
