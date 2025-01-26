# Kubernetes the hard way (with raspberry pi and docker)

This tutorial uses the [kubernetes-the-hard-way](https://github.com/kelseyhightower/kubernetes-the-hard-way) repo by [kelseyhightower](https://github.com/kelseyhightower) as reference. Some of the steps will be an exact replica of kelseyhightower's, but I will try my best to guide you through the process of setting up containers in various hosts (`EC2` and `Raspberry PI`).

## Labs

This tutorial uses 3 hosts: two EC2 containers, and 1 Raspberry Pi. This will provide a good starting point to connect different nodes from different hosts in different environments (cloud & local machine).

|     | Machine      | OS             | Purpose                    |
| --- | ------------ | -------------- | -------------------------- |
| 1   | EC2          | Ubuntu         | K8s Server & Control Plane |
| 2   | EC2          | Ubuntu         | Node-0                     |
| 3   | Raspberry Pi | PI OS (64 bit) | Node-1                     |

- [01 - Overview](https://github.com/Jaecom/kubernetes-the-hard-way-raspberrypi-docker/blob/main/docs/01-overview.md)
- [02 - EC2 Setup](https://github.com/Jaecom/kubernetes-the-hard-way-raspberrypi-docker/blob/main/docs/02-ec2-setup.md)
- [03 - Pi Setup](https://github.com/Jaecom/kubernetes-the-hard-way-raspberrypi-docker/blob/main/docs/03-raspberry-pi-setup.md)
- [04 - Running Docker Containers](https://github.com/Jaecom/kubernetes-the-hard-way-raspberrypi-docker/blob/main/docs/04-docker-container-setup.md)
- [05 - Wireguard VPN Setup](https://github.com/Jaecom/kubernetes-the-hard-way-raspberrypi-docker/blob/main/docs/05-wireguard-setup.md)
- [06 - Jumpbox](https://github.com/Jaecom/kubernetes-the-hard-way-raspberrypi-docker/blob/main/docs/06-jumpbox.md)
- [07 - Provisioning Resources](https://github.com/Jaecom/kubernetes-the-hard-way-raspberrypi-docker/blob/main/docs/07-compute-resources.md)
- [07 - Provisioning Resources (SSH Config)](https://github.com/Jaecom/kubernetes-the-hard-way-raspberrypi-docker/blob/main/docs/07-compute-resources-ssh-config.md)
- [08 - Generating Certificates](https://github.com/Jaecom/kubernetes-the-hard-way-raspberrypi-docker/blob/main/docs/08-certificate-authority.md)
- [09 - Setting up Kubernetes Config](https://github.com/Jaecom/kubernetes-the-hard-way-raspberrypi-docker/blob/main/docs/09-kubernetes-configuration-files.md)
- [10 - Setting Up Encryption](https://github.com/Jaecom/kubernetes-the-hard-way-raspberrypi-docker/blob/main/docs/10-data-encryption-keys.md)
- [11 - Bootstrapping etcd](https://github.com/Jaecom/kubernetes-the-hard-way-raspberrypi-docker/blob/main/docs/11-bootstrapping-etcd.md)
- [12 - Setting Up Kubernetes Controllers](https://github.com/Jaecom/kubernetes-the-hard-way-raspberrypi-docker/blob/main/docs/12-bootstrapping-kubernetes-controllers.md)
- [13 - Setting Up Worker Nodes](https://github.com/Jaecom/kubernetes-the-hard-way-raspberrypi-docker/blob/main/docs/13-bootstrapping-kubernetes-workers.md)
- [14 - Setting Up Kubectl](https://github.com/Jaecom/kubernetes-the-hard-way-raspberrypi-docker/blob/main/docs/14-configuring-kubectl.md)
- [15 - Configuring Pod Networks](https://github.com/Jaecom/kubernetes-the-hard-way-raspberrypi-docker/blob/main/docs/15-pod-network-routes.md)
- [16 - Smoke Test](https://github.com/Jaecom/kubernetes-the-hard-way-raspberrypi-docker/blob/main/1docs/16-smoke-test.md)
- [17 - Cleaning Up](https://github.com/Jaecom/kubernetes-the-hard-way-raspberrypi-docker/blob/main/docs/17-cleanup.md)
