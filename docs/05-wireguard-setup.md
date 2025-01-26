# Wireguard VPN Setup

If you have no background in Wireguard VPN, here is a good reference to start with: [Set Up a WireGuard VPN on Linux](https://medium.com/@jae.yoon7373/how-to-manually-configure-a-vpn-with-wireguard-with-linux-781f5f106f8b)

This is a brief overview of the IP Addresses we are going to assign to each container:

| Host         | Container | IP Address | Subnet | Description | Listen Port |
| ------------ | --------- | ---------- | ------ | ----------- | ----------- |
| EC2 A        | `server`  | 10.0.0.1   | /24    | Hub         | 51820/udp   |
| EC2 A        | `jumpbox` | 10.0.0.2   | /24    | Peer        | -           |
| EC2 B        | `node-0`  | 10.0.1.1   | /24    | Peer        | -           |
| Raspberry Pi | `node-1`  | 10.0.2.1   | /24    | Peer        | -           |

> [!Note]
> The `server` container in `EC2 A` will act as a central hub for the VPN network. It will forward ip traffic so that the different containers are able to communicate with one another.

## Generate Private and Public Key

Ceate a `private.key` and `public.key` in each of the 4 containers. You will need 4 sets of keys for each container.

```
# In your server container
wg genkey | tee wireguard_server_private.key | wg pubkey > wireguard_server_public.key

# In your jumpbox container
wg genkey | tee wireguard_jumpbox_private.key | wg pubkey > wireguard_jumpbox_public.key

# In your node-0 container
wg genkey | tee wireguard_node-0_private.key | wg pubkey > wireguard_node-0_public.key

# In your node-1 container
wg genkey | tee wireguard_node-1_private.key | wg pubkey > wireguard_node-1_public.key
```

```
# In your server container
echo "Server Private Key: $(cat wireguard_server_private.key)"
echo "Server Public Key: $(cat wireguard_server_public.key)"

# In your jumpbox container
echo "Jumpbox Private Key: $(cat wireguard_jumpbox_private.key)"
echo "Jumpbox Public Key: $(cat wireguard_jumpbox_public.key)"

# In your node-0 container
echo "Node-0 Private Key: $(cat wireguard_node-0_private.key)"
echo "Node-0 Public Key: $(cat wireguard_node-0_public.key)"

# In your node-1 container
echo "Node-1 Private Key: $(cat wireguard_node-1_private.key)"
echo "Node-1 Public Key: $(cat wireguard_node-1_public.key)"

```

## Wireguard Config

### Server Container

In your Server EC2 Container `ssh` into your `server` docker container:

```
ssh root@localhost -p 2223
# Password is admin
```

Create a wireguard config file at `/etc/wireguard/wg0.conf`

```
nano /etc/wireguard/wg0.conf
```

Save the following config using your generated keys above or the keys below:

```
[Interface]
Address = 10.0.0.1/24
ListenPort = 51820
PrivateKey = eD5rkbHBXJoMkTDGOwACYlRe/zMDj2dZthwXXnCdI1g= # Server Private Key

# Jumpbox
[Peer]
PublicKey = 2I9u5kruJKnsqyEigq7oVDeu6svG8XrNFWoDMhH++ls= # Jumpbox Public Key
AllowedIPs = 10.0.0.2/32

# Node-0
[Peer]
PublicKey = 7lEdXmpAJd6ObiZBogaBObbhu8UVwQTObXPWUMA+tnQ= # Node-0 Private Key
AllowedIPs = 10.0.1.1/32, 10.244.1.0/24

# Node-1
[Peer]
PublicKey = 3XageH4zmDPUfzLDbzuPT+epMNYdOep21oEFvuar82M= # Node-1 Private Key
AllowedIPs = 10.0.2.1/32, 10.244.2.0/24
```

Start the wireguard interface:

```
wg-quick up wg0
```

You need to repeat this process for the `jumpbox`, `node-0`, and `node-1` container.

## Jumpbox Container

```
[Interface]
Address = 10.0.0.2/24
PrivateKey = cA3M6UOVlTIdaukDGmcL/+QIZoYv7dnSCaYxlaRkIX8=

[Peer]
PublicKey = kyHQsgAl68A3s7f4BJb2BsDtEGBdG4MIDyJmlGu9GyE=
Endpoint = <EC2 A Public IP Address>:51820
AllowedIPs = 10.0.0.0/16, 10.244.0.0/16
PersistentKeepalive = 25
```

```
wg-quick up wg0
```

## Node-0 Container

```
[Interface]
Address = 10.0.1.1/24
PrivateKey = QD6VPu6ZfU6IpOghkLFZpDoh3uofAJ0MLwKDpfHUh1o=

[Peer]
PublicKey = kyHQsgAl68A3s7f4BJb2BsDtEGBdG4MIDyJmlGu9GyE=
Endpoint = <EC2 A Public IP Address>:51820
AllowedIPs = 10.0.0.0/16, 10.244.0.0/16
PersistentKeepalive = 25
```

```
wg-quick up wg0
```

## Node-1 Container

```
[Interface]
Address = 10.0.2.1/24
PrivateKey = IA4tXR3v+WFuSMx71islv+R8/VdOyGljcc7rlRH7DEo=

[Peer]
PublicKey = kyHQsgAl68A3s7f4BJb2BsDtEGBdG4MIDyJmlGu9GyE=
Endpoint = <EC2 A Public IP Address>:51820
AllowedIPs = 10.0.0.0/16, 10.244.0.0/16
PersistentKeepalive = 25
```

```
wg-quick up wg0
```

## Checking WireGuard VPN Connection

In your `server` Container, check that wireguard has successfully connected all other three containers:

```
wg
```

You should see something like this:

```
interface: wg0
  public key: kyHQsgAl68A3s7f4BJb2BsDtEGBdG4MIDyJmlGu9GyE=
  private key: (hidden)
  listening port: 51820

# Jumpbox
peer: 2I9u5kruJKnsqyEigq7oVDeu6svG8XrNFWoDMhH++ls=
  endpoint: <Jumpbox IP>:52418
  allowed ips: 10.0.0.2/32
  latest handshake: 11 seconds ago
  transfer: 963.05 MiB received, 14.63 MiB sent

# Node-0
peer: 7lEdXmpAJd6ObiZBogaBObbhu8UVwQTObXPWUMA+tnQ=
  endpoint: <Node-0 IP>:40178
  allowed ips: 10.0.1.1/32, 10.244.1.0/24
  latest handshake: 29 seconds ago
  transfer: 16.20 MiB received, 343.85 MiB sent

# Node-1
peer: 3XageH4zmDPUfzLDbzuPT+epMNYdOep21oEFvuar82M=
  endpoint: <Node-1 IP>:47527
  allowed ips: 10.0.2.1/32, 10.244.2.0/24
  latest handshake: 55 seconds ago
  transfer: 25.06 MiB received, 330.67 MiB sent

```

## Enable IP Forwarding

Enable IP Forwarding in the `server` container firewall rules:

```
sudo nano /etc/sysctl.conf
```

Uncomment `net.ipv4.ip_forward = 1` part of the config file:

```
...

# Uncomment the next line to enable TCP/IP SYN cookies
# See http://lwn.net/Articles/277146/
# Note: This may impact IPv6 TCP sessions too
#net.ipv4.tcp_syncookies=1

# Uncomment the next line to enable packet forwarding for IPv4
net.ipv4.ip_forward=1

...
```

Apply the changes

```
sudo sysctl -p
```

Add the following firegwall rule to forward ip:

```
iptables -A FORWARD -i wg0 -o wg0 -s 10.0.0.0/16 -d 10.0.0.0/16 -j ACCEPT
```

Ensure firewall persist across reboots:

```
apt install netfilter-persistent
netfilter-persistent save
netfilter-persistent reload
systemctl enable netfilter-persistent
```

## Checking

From any container, you should be able to ping any other container:

```
ping 10.0.0.1
ping 10.0.0.2
ping 10.0.1.1
ping 10.0.2.1
```

Next: [06 - Jumpbox](https://github.com/Jaecom/kubernetes-the-hard-way-raspberrypi-docker/blob/main/docs/06-jumpbox.md)
