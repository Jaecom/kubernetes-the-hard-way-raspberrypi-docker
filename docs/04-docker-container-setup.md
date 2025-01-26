# Setting Up Docker Containers

## Create base image

This step should be done for your `Server EC2`, `Node-0 EC2`, and `Raspberry Pi`.

In you desired directory, copy the [debian-bookworm-ssh](https://github.com/Jaecom/kubernetes-the-hard-way-raspberrypi-docker/blob/main/debian-bookworm-ssh) dockerfile to create the base image the containers are going to run on. It will have ssh enabled by default.

> [!Note]
> devian-bookworm-ssh will configure SSH to be run on port 6001.

Create the debian-bookworm-ssh dockerfile:

```
vim debian-bookworm-ssh
```

This dockerfile uses the `debian:bookworm` as a base image and installs necessary packages. The default password is `admin`.

After copy and pasting the contents from [debian-bookworm-ssh](https://github.com/Jaecom/kubernetes-the-hard-way-raspberrypi-docker/blob/main/debian-bookworm-ssh), build the image:

```
sudo docker build -t debian-bookworm-ssh -f debian-bookworm-ssh .
```

You should be able to see your image successfully built.

```
sudo docker image ls
```

```
REPOSITORY            TAG       IMAGE ID       CREATED        SIZE
debian-bookworm-ssh   latest    366c550a91c4   1 minute ago   208MB
```

## Running Containers

### EC2 #1

In your `EC2 #1` run the `jumpbox` container and `server` containers.

```
# Run jumpbox container
sudo docker run \
--name jumpbox -d \
-p 2222:6001 \
-p 8080:8080 \
--privileged \
debian-bookworm-ssh

# Run server container
sudo docker run \
--name server -d \
-p 2223:6001 \
-p 6443:6443 \
-p 51820:51820/udp \
--privileged \
debian-bookworm-ssh
```

### EC2 #2

```
# Node-0 docker container in Host #1
sudo docker run \
--name node-0 -d \
--network host \
--privileged \
-v /lib/modules/$(uname -r):/lib/modules/$(uname -r) \
-v /mnt/node-0/containerd/:/var/lib/containerd \
-v /mnt/node-0/kubelet:/var/lib/kubelet \
debian-bookworm-ssh
```

### Raspberry Pi

```
sudo docker run \
--name node-1 -d \
--network host \
--privileged \
-v /lib/modules/$(uname -r):/lib/modules/$(uname -r) \
-v /mnt/node-1/containerd/:/var/lib/containerd \
-v /mnt/node-1/kubelet:/var/lib/kubelet \
debian-bookworm-ssh
```

> [!Note]
> Node containers requires a volume mount for the `containerd` and `kubelet` directories. This is to prevent nested file overlays which will cause an error if not mounted. You can change the mount to be somewhere else by changing the `/mnt/node-0/containerd` and `/mnt/node-0/kubelet` paths.

## Checking Containers

Check that the containers are running:

```
sudo docker ps
```

You should see something like this:

`Server EC2`

```
CONTAINER ID   IMAGE                 COMMAND                  CREATED          STATUS          PORTS                                                                                                                               NAMES
1b5aeeeaebf3   debian-bookworm-ssh   "/lib/systemd/systemd"   34 seconds ago   Up 34 seconds   0.0.0.0:6443->6443/tcp, :::6443->6443/tcp, 0.0.0.0:51820->51820/udp, :::51820->51820/udp, 0.0.0.0:2223->22/tcp, :::2223->6001/tcp   server
5f9b4249a82a   debian-bookworm-ssh   "/lib/systemd/systemd"   34 seconds ago   Up 34 seconds   0.0.0.0:8080->8080/tcp, :::8080->8080/tcp, 0.0.0.0:2222->22/tcp, :::2222->6001/tcp                                                  jumpbox
```

`Node-0 EC2`

```
CONTAINER ID   IMAGE                 COMMAND                  CREATED          STATUS          PORTS     NAMES
b4caefd8a76d   debian-bookworm-ssh   "/lib/systemd/systemd"   50 seconds ago   Up 50 seconds             node-0
```

`Node-1 EC2`

```
CONTAINER ID   IMAGE                 COMMAND                  CREATED          STATUS          PORTS     NAMES
3a587de448e2   debian-bookworm-ssh   "/lib/systemd/systemd"   51 seconds ago   Up 51 seconds             node-1
```

Next: [05 - Wireguard Setup](https://github.com/Jaecom/kubernetes-the-hard-way-raspberrypi-docker/blob/main/docs/05-wireguard-setup.md)
