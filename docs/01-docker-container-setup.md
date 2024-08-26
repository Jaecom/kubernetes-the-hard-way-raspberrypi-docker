# Setting Up Docker Containers

We will first set up your raspberry pi to run the 4 base containers required: `jumpbox`, `server`, `node-0`,`node-1`

## Create base image

Create a debian-bookworm-ssh dockerfile to create the base image the containers are going to run on. It will have ssh enabled by default:

```
vim debian-bookworm-ssh
```

```
# Use the Debian Bookworm base image
FROM debian:bookworm

# Install necessary packages including systemd and OpenSSH server
RUN apt-get update && \
    apt-get install -y systemd systemd-sysv openssh-server && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*

# Create necessary directory for the SSH daemon to run
RUN mkdir -p /var/run/sshd

# Set root password (replace 'admin' with a password)
RUN echo 'root:admin' | chpasswd

# Allow root login (optional and not recommended for production)
RUN sed -i 's/#PermitRootLogin prohibit-password/PermitRootLogin yes/' /etc/ssh/sshd_config

# SSH login fix. Otherwise user is kicked off after login
RUN sed -i 's@session\s*required\s*pam_loginuid.so@session optional pam_loginuid.so@g' /etc/pam.d/sshd

# Enable SSH service
RUN systemctl enable ssh

# Expose SSH port
EXPOSE 22

# Use systemd as the init system
CMD ["/lib/systemd/systemd"]
```

```
sudo docker build -t debian-bookworm-ssh -f debian-bookworm-ssh .
```

```
sudo docker image ls
```

You should be able to see your image successfully built.

```
REPOSITORY            TAG       IMAGE ID       CREATED        SIZE
debian-bookworm-ssh   latest    366c550a91c4   1 minute ago   208MB
```

<br/>

## Run the containers

Run the jumpbox container and server containers.

```
# Run jumpbox container
sudo docker run \
--name jumpbox -d \
-p 2222:22 \
-p 8080:8080 \
--privileged \
debian-bookworm-ssh

# Run server container
sudo docker run \
--name server -d \
-p 2223:22 \
-p 6443:6443 \
--privileged \
debian-bookworm-ssh
```

Run the node containers. Node containers requires a volume mount for the `containerd` and `kubelet` directories. This is to prevent nested file overlays which will cause an error if not mounted. You can change the mount to be somewhere else by changing the `/mnt/node-0/containerd` and `/mnt/node-0/kubelet` paths.

```
# Run node-0 container
sudo docker run \
--name node-0 -d \
-p 2224:22 \
--privileged \
-v /lib/modules/$(uname -r):/lib/modules/$(uname -r) \
-v /mnt/node-0/containerd/:/var/lib/containerd \
-v /mnt/node-0/kubelet:/var/lib/kubelet \
debian-bookworm-ssh

# Run node-1 container
sudo docker run \
--name node-1 -d \
-p 2225:22 \
--privileged \
-v /lib/modules/$(uname -r):/lib/modules/$(uname -r) \
-v /mnt/node-1/containerd/:/var/lib/containerd \
-v /mnt/node-1/kubelet:/var/lib/kubelet \
debian-bookworm-ssh
```

Check that the containers are running:

```
sudo docker ps
```

You should see something like this:

```
CONTAINER ID   IMAGE                 COMMAND                  CREATED      STATUS      PORTS                                                                              NAMES
a63f0b745835   debian-bookworm-ssh   "/lib/systemd/systemd"   4 days ago   Up 4 days   0.0.0.0:2225->22/tcp, :::2225->22/tcp                                              node-1
b88fa97d1896   debian-bookworm-ssh   "/lib/systemd/systemd"   4 days ago   Up 4 days   0.0.0.0:2224->22/tcp, :::2224->22/tcp                                              node-0
de9ab495f71d   debian-bookworm-ssh   "/lib/systemd/systemd"   4 days ago   Up 4 days   0.0.0.0:6443->6443/tcp, :::6443->6443/tcp, 0.0.0.0:2223->22/tcp, :::2223->22/tcp   server
4e7ebeaaeeea   debian-bookworm-ssh   "/lib/systemd/systemd"   4 days ago   Up 4 days   0.0.0.0:8080->8080/tcp, :::8080->8080/tcp, 0.0.0.0:2222->22/tcp, :::2222->22/tcp   jumpbox
```

Next: [Setting up Jumpbox](https://github.com/Jaecom/kubernetes-the-hard-way-raspberrypi-docker/blob/main/docs/02-jumpbox.md)
