# Smoke Test

> [!NOTE]
> In this section, you can follow the steps without any changes in kelseyhightower's official guide: [12-smoke-test.md](https://github.com/kelseyhightower/kubernetes-the-hard-way/blob/master/docs/12-smoke-test.md). I added the shortened version of the code below for convenience.

## Data Encryption

The following guide will be run in your `jumpbox` container.

Install `bsdmainutils` package in your `server` container:

```
ssh root@server 'apt update && apt install -y bsdmainutils'
```

Create a generic secret:

```
kubectl create secret generic kubernetes-the-hard-way \
 --from-literal="mykey=mydata"

```

Print a hexdump of the kubernetes-the-hard-way secret stored in etcd:

```

ssh root@server \
 'etcdctl get /registry/secrets/default/kubernetes-the-hard-way | hexdump -C'

```

```
00000000  2f 72 65 67 69 73 74 72  79 2f 73 65 63 72 65 74  |/registry/secret|
00000010  73 2f 64 65 66 61 75 6c  74 2f 6b 75 62 65 72 6e  |s/default/kubern|
00000020  65 74 65 73 2d 74 68 65  2d 68 61 72 64 2d 77 61  |etes-the-hard-wa|
00000030  79 0a 6b 38 73 3a 65 6e  63 3a 61 65 73 63 62 63  |y.k8s:enc:aescbc|
00000040  3a 76 31 3a 6b 65 79 31  3a 67 0e 3d 21 19 94 e1  |:v1:key1:g.=!...|
00000050  56 92 3f ed 27 8a 1e ee  78 4e 3e 61 a5 40 6a 3b  |V.?.'...xN>a.@j;|
00000060  71 62 0d b2 50 76 69 0c  f8 4e d6 b5 5c 24 11 c3  |qb..Pvi..N..\$..|
00000070  e5 3c 16 ae 42 5f 00 9e  44 0d cf fd bb b0 e0 14  |.<..B_..D.......|
00000080  d6 a0 a9 0d 5c 8d 09 5e  78 5c a8 ae 96 7a ca 42  |....\..^x\...z.B|
00000090  19 b3 3e 67 d6 74 3a 1b  4a 35 40 94 36 cc 2b 81  |..>g.t:.J5@.6.+.|
000000a0  e6 48 61 ac 2e 78 c6 8b  df b7 66 a4 d6 20 cc 88  |.Ha..x....f.. ..|
000000b0  25 db 00 6b b5 59 68 e4  6d 55 0f 77 2a 9c 9c bf  |%..k.Yh.mU.w*...|
000000c0  92 09 d4 45 c7 df 87 1b  a8 92 2f af 9a 3c 6b 9f  |...E....../..<k.|
000000d0  5e 0a 94 32 c3 53 07 37  31 93 3a 0d be 9c dc 05  |^..2.S.71.:.....|
000000e0  6b 3b a0 50 f6 9b c5 60  5a 7c 34 b4 52 01 d6 b3  |k;.P...`Z|4.R...|
000000f0  f1 7b c9 82 5b ee 71 fc  0b 16 d8 b9 6b 42 53 27  |.{..[.q.....kBS'|
00000100  e1 96 d4 e9 6d b4 6e d0  65 75 3f d3 d6 66 4d 24  |....m.n.eu?..fM$|
00000110  82 9f dc 3c d3 fb 2c eb  38 4a 9e c7 01 6e bb 69  |...<..,.8J...n.i|
00000120  db 27 2e ef 72 0c 71 b5  4f 1b a0 37 64 60 b7 d8  |.'..r.q.O..7d`..|
00000130  dc 58 e5 34 57 fa 8d 48  a1 1e 87 43 94 f7 54 42  |.X.4W..H...C..TB|
00000140  14 6c 0b 0e 13 b1 81 bc  cf 9a ac 3b bd 8d 65 41  |.l.........;..eA|
00000150  c6 90 11 46 34 0d 79 c8  e6 0a                    |...F4.y...|
0000015a
```

## Deployments

Create a deployment for the nginx web server:

```
kubectl create deployment nginx \
 --image=nginx:latest
```

List the pod created by the nginx deployment:

```
kubectl get pods -l app=nginx
```

```
NAME READY STATUS RESTARTS AGE
nginx-56fcf95486-c8dnx 1/1 Running 0 8s
```

## Port Forwarding

Retrieve the full name of the nginx pod:

```
POD_NAME=$(kubectl get pods -l app=nginx \
 -o jsonpath="{.items[0].metadata.name}")
```

Forward port 8080 on your local machine to port 80 of the nginx pod:

```
kubectl port-forward $POD_NAME 8080:80
```

```
Forwarding from 127.0.0.1:8080 -> 80
Forwarding from [::1]:8080 -> 80
```

In a new terminal make an HTTP request using the forwarding address:

```
curl --head http://127.0.0.1:8080
```

```
HTTP/1.1 200 OK
Server: nginx/1.25.3
Date: Sun, 29 Oct 2023 01:44:32 GMT
Content-Type: text/html
Content-Length: 615
Last-Modified: Tue, 24 Oct 2023 13:46:47 GMT
Connection: keep-alive
ETag: "6537cac7-267"
Accept-Ranges: bytes
```

## Logs

Print the nginx pod logs:

```
kubectl logs $POD_NAME
```

```
...
127.0.0.1 - - [01/Nov/2023:06:10:17 +0000] "HEAD / HTTP/1.1" 200 0 "-" "curl/7.88.1" "-"
```

## Exec

```
kubectl exec -ti $POD_NAME -- nginx -v
```

```
nginx version: nginx/1.25.3
```

## Services

Expose the nginx deployment using a NodePort service:

```
kubectl expose deployment nginx \
 --port 80 --type NodePort
```

Retrieve the node port assigned to the nginx service:

```
NODE_PORT=$(kubectl get svc nginx \
 --output=jsonpath='{range .spec.ports[0]}{.nodePort}')
```

Make an HTTP request using the IP address and the nginx node port:

```
curl -I http://node-0:${NODE_PORT}
```

```
HTTP/1.1 200 OK
Server: nginx/1.25.3
Date: Sun, 29 Oct 2023 05:11:15 GMT
Content-Type: text/html
Content-Length: 615
Last-Modified: Tue, 24 Oct 2023 13:46:47 GMT
Connection: keep-alive
ETag: "6537cac7-267"
Accept-Ranges: bytes
```

Next: [Cleaning Up](https://github.com/Jaecom/kubernetes-the-hard-way-raspberrypi-docker/blob/main/docs/13-cleanup.md)
