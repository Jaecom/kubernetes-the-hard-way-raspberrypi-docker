# Cleaning Up

## Stop & remove the docker containers

### EC2 A

```
sudo docker stop jumpbox server
sudo docker remove jumpbox server
```

### EC2 B

```
sudo docker stop node-0
sudo docker remove node-0
```

### Raspberry Pi

```
sudo docker stop node-1
sudo docker remove node-1
```

## Delete the directory files

Remove the `/mnt` directory from the node hosts:

### EC2 B

```
sudo rm -rf /mnt
```

### Raspberry Pi

```
sudo rm -rf /mnt
```

## Delete the EC2 Containers

If you have creates the `EC2 A` and `EC2 B` just for the purposes of this guide, terminate them using the AWS Console.
