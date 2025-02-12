# Setting up EC2

## Create EC2 Containers

Create 2 EC2 containers with ubuntu (preferrably 24.02 or higher)

| Name  | Type     | OS     | Version | Purpose                           |
| ----- | -------- | ------ | ------- | --------------------------------- |
| EC2 A | t2.micro | Ubuntu | 24.02   | Kubernetes Server & Control Plane |
| EC2 B | t2.micro | Ubuntu | 24.02   | Kubernetes Node                   |

We will refer to each EC2 instance as the `EC2 A` or the `EC2 B`.

## Edit Security Group

Edit the Security Group of each instance to allow inbound rules of the following port:

| Name  | Security Group Inbound Rule                             |
| ----- | ------------------------------------------------------- |
| EC2 A | `51820/udp` `2222/tcp` `2223/tcp` `6443/tcp` `8080/tcp` |
| EC2 B | `31703/tcp` `6001/tcp`                                  |

> [!IMPORTANT]
> From now on, we’ll refer to the two EC2 instances as `EC2 A` (control plane) and `EC2 B` (node). Make sure you run each command on the correct instance.

## Install Docker

Docker
You must have docker installed in your EC2 Instances. In case of a `Ubuntu`, use the official docs: [Install Docker Engine on Ubuntu](https://docs.docker.com/engine/install/ubuntu/)

<!-- Add Command to Create EC2 Container and open ports -->

Next: [03 - Raspberry PI Setup](https://github.com/Jaecom/kubernetes-the-hard-way-raspberrypi-docker/blob/main/docs/03-raspberry-pi-setup.md)
