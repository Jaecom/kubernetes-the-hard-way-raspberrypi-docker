# Provisioning Pod Network Routes

## The Routing Table

From your jumpbox machine,

SSH into `node-0`:

```
ssh node-0
```

```
iptables -A FORWARD -i wg0 -o cni0 -s 10.0.0.0/16 -d 10.244.1.0/24 -j ACCEPT
```

SSH into `node-1`:

```
ssh node-1
```

```
iptables -A FORWARD -i wg0 -o cni0 -s 10.0.0.0/16 -d 10.244.2.0/24 -j ACCEPT
```

> [!Note]
> These commands modify the firewall settings to allow traffic from the WireGuard interface `wg0` to the container network interface `cni0`. This ensures that network packets can pass between the WireGuard network `10.0.0.0/16` and the Kubernetes pod networks on each node - `10.244.1.0/24` or `10.244.2.0/24`.

Next: [16 - Testing](https://github.com/Jaecom/kubernetes-the-hard-way-raspberrypi-docker/blob/main/docs/16-smoke-test.md)
