# Kubernetes the hard way (with raspberry pi and docker)

This tutorial uses the [kubernetes-the-hard-way](https://github.com/kelseyhightower/kubernetes-the-hard-way) repo by [kelseyhightower](https://github.com/kelseyhightower) as reference. Some of the steps will be an exact replica of kelseyhightower's, but I will try my best to guide you through the process of setting up containers inside your raspberry pi (or any host machine).

## Labs

This tutorial uses 4 docker containers in a linux based machine. In this case, it will use a raspberry pi 5 with the Raspberry PI OS (64bit) which is a port of Debian Bookworm.

- [00 - Pi Setup](https://github.com/Jaecom/kubernetes-the-hard-way-raspberrypi-docker/blob/main/docs/00-raspberry-pi-setup.md)
- [01 - Running Docker Containers](https://github.com/Jaecom/kubernetes-the-hard-way-raspberrypi-docker/blob/main/docs/01-docker-container-setup.md)
- [02 - Jumpbox](https://github.com/Jaecom/kubernetes-the-hard-way-raspberrypi-docker/blob/main/docs/02-jumpbox.md)
- [03 - Provisioning Resources](https://github.com/Jaecom/kubernetes-the-hard-way-raspberrypi-docker/blob/main/docs/03-compute-resources.md)
- [04 - Generating Certificates](https://github.com/Jaecom/kubernetes-the-hard-way-raspberrypi-docker/blob/main/docs/04-certificate-authority.md)
- [05 - Kubernetes Configuration](https://github.com/Jaecom/kubernetes-the-hard-way-raspberrypi-docker/blob/main/docs/05-kubernetes-configuration-files.md)
- [06 - Date Encryption](https://github.com/Jaecom/kubernetes-the-hard-way-raspberrypi-docker/blob/main/docs/06-data-encryption-keys.md)
- [07 - Bootstrapping etcd](https://github.com/Jaecom/kubernetes-the-hard-way-raspberrypi-docker/blob/main/docs/07-bootstrapping-etcd.md)
- [08 - Kubernetes Controllers](https://github.com/Jaecom/kubernetes-the-hard-way-raspberrypi-docker/blob/main/docs/08-bootstrapping-kubernetes-controllers.md)
- [09 - Kubernetes Worker Nodes](https://github.com/Jaecom/kubernetes-the-hard-way-raspberrypi-docker/blob/main/docs/09-bootstrapping-kubernetes-workers.md)
- [10 - Kubectl](https://github.com/Jaecom/kubernetes-the-hard-way-raspberrypi-docker/blob/main/docs/10-configuring-kubectl.md)
- [11 - Pod Networks](https://github.com/Jaecom/kubernetes-the-hard-way-raspberrypi-docker/blob/main/docs/11-pod-network-routes.md)
- [12 - Testing](https://github.com/Jaecom/kubernetes-the-hard-way-raspberrypi-docker/blob/main/docs/12-smoke-test.md)
- [13 - Cleanup](https://github.com/Jaecom/kubernetes-the-hard-way-raspberrypi-docker/blob/main/docs/13-cleanup.md)
