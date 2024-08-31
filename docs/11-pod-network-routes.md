# Provisioning Pod Network Routes

> [!NOTE]
> In this section, you can follow the steps without any changes in kelseyhightower's official guide: [10-configuring-kubectl.md](https://github.com/kelseyhightower/kubernetes-the-hard-way/blob/master/docs/10-configuring-kubectl.md). I added the shortened version of the code below for convenience.

## The Routing Table

Add the IP addresses:

```
{
  SERVER_IP=$(grep server machines.txt | cut -d " " -f 1)
  NODE_0_IP=$(grep node-0 machines.txt | cut -d " " -f 1)
  NODE_0_SUBNET=$(grep node-0 machines.txt | cut -d " " -f 4)
  NODE_1_IP=$(grep node-1 machines.txt | cut -d " " -f 1)
  NODE_1_SUBNET=$(grep node-1 machines.txt | cut -d " " -f 4)
}

ssh root@server <<EOF
  ip route add ${NODE_0_SUBNET} via ${NODE_0_IP}
  ip route add ${NODE_1_SUBNET} via ${NODE_1_IP}
EOF

ssh root@node-0 <<EOF
  ip route add ${NODE_1_SUBNET} via ${NODE_1_IP}
EOF

ssh root@node-1 <<EOF
  ip route add ${NODE_0_SUBNET} via ${NODE_0_IP}
EOF
```

## Verification

```
ssh root@server ip route
ssh root@node-0 ip route
ssh root@node-1 ip route
```

```
default via XXX.XXX.XXX.XXX dev ens160
10.200.0.0/24 via XXX.XXX.XXX.XXX dev ens160
10.200.1.0/24 via XXX.XXX.XXX.XXX dev ens160
XXX.XXX.XXX.0/24 dev ens160 proto kernel scope link src XXX.XXX.XXX.XXX

default via XXX.XXX.XXX.XXX dev ens160
10.200.1.0/24 via XXX.XXX.XXX.XXX dev ens160
XXX.XXX.XXX.0/24 dev ens160 proto kernel scope link src XXX.XXX.XXX.XXX

default via XXX.XXX.XXX.XXX dev ens160
10.200.0.0/24 via XXX.XXX.XXX.XXX dev ens160
XXX.XXX.XXX.0/24 dev ens160 proto kernel scope link src XXX.XXX.XXX.XXX
```

Next: [Testing](https://github.com/Jaecom/kubernetes-the-hard-way-raspberrypi-docker/blob/main/docs/12-smoke-test.md)
