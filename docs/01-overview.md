# Overview

## Docker Containers

Here is a more detaile overview of the setup of this guide:

|     | Machine      | OS     | Ports Used                                              |
| --- | ------------ | ------ | ------------------------------------------------------- |
| 1   | EC2          | Ubuntu | `51820/udp` `2222/tcp` `2223/tcp` `6443/tcp` `8080/tcp` |
| 2   | EC2          | Ubuntu | `31703/tcp` `6001/tcp`                                  |
| 3   | Raspberry Pi | PI OS  | `31703/tcp` `6002/tcp`                                  |

This is a high level overview of the opened ports that should be accessible. We will go over the configuration in each lab section.

## VPC vs VPN

To connect the 3 machines together, rather than use VPC, we will use a VPN with the `wireguard` package. Configuration will be guided step by step.

More information about Wireguard can be found in the [WireGuard Docs](https://www.wireguard.com/)

Next: [02 - EC2 Setup](https://github.com/Jaecom/kubernetes-the-hard-way-raspberrypi-docker/blob/main/docs/02-ec2-setup.md)
