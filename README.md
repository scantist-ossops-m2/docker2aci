# docker2aci - Convert docker images to ACI

docker2aci is a small library and CLI binary that converts Docker images to
[ACI][aci]. It takes as input either a file generated by "docker save" or a
Docker registry URL. It gets all the layers of a Docker image and squashes them
into an ACI image. Optionally, it can generate one ACI for each layer, setting
the correct dependencies.

All ACIs generated are compressed with gzip.


## Build

Installation is simple as:

	go get github.com/appc/docker2aci

or as involved as:

	git clone git://github.com/appc/docker2aci
	cd docker2aci
	go get -d ./...
	go build

## Volumes

Docker Volumes get converted to mountPoints in the [Image Manifest
Schema][imageschema]. Since mountPoints need a name and Docker Volumes don't,
docker2aci generates a name by appending the path to `volume-` replacing
non-alphanumeric characters with dashes. That is, if a Volume has `/var/tmp`
as path, the resulting mountPoint name will be `volume-var-tmp`.

When the docker2aci CLI binary converts a Docker Volume to a mountPoint it will
print its name, path and whether it is read-only or not.

## Ports

Docker Ports get converted to ports in the [Image Manifest
Schema][imageschema]. The resulting port name will be the port number and the
protocol separated by a dash. For example: `6379-tcp`.

## CLI examples

```
$ docker2aci docker://busybox
Downloading cf2616975b4a: [====================================] 32 B/32 B
Downloading 6ce2e90b0bc7: [====================================] 1.15 MB/1.15 MB
Downloading 8c2e06607696: [====================================] 32 B/32 B

Generated ACI(s):
busybox-latest.aci
$ actool --debug validate busybox-latest.aci
busybox-latest.aci: valid app container image
```

```
$ /docker2aci --nosquash docker://quay.io/coreos/etcd:latest
Downloading 3c79dd31bf84: [====================================] 85 B/85 B
Downloading 8423185475fe: [====================================] 4 MB/4 MB
Downloading 185eec9979eb: [====================================] 2.02 MB/2.02 MB
Downloading 78d63abf03b9: [====================================] 23 B/23 B
Downloading c5f34efc4446: [====================================] 23 B/23 B

Generated ACI(s):
coreos-etcd-3c79dd31bf84b2fb7c55354f5069964a72bb6ae0c1263331c0f83ce4c32a4b6a-latest-linux-amd64.aci
coreos-etcd-8423185475fe5bb0c86dc98ba2816ca9cc29cbf3ec5f3ec091963854746ee131-latest-linux-amd64.aci
coreos-etcd-185eec9979eb1288f1412ec997860d3c865ac6a9e5c71487d9876bc0ec7bbdfe-latest-linux-amd64.aci
coreos-etcd-78d63abf03b980919deaac3454a80496559da893948f427868492fa8a0d717ab-latest-linux-amd64.aci
coreos-etcd-c5f34efc44466ec7abb9a68af20d2f876ea691095747e7e5a62e890cdedadcdc-latest-linux-amd64.aci
```

```
$ docker save -o ubuntu.docker ubuntu
$ docker2aci ubuntu.docker
Extracting 706766fe1019
Extracting a62a42e77c9c
Extracting 2c014f14d3d9
Extracting b7cf8f0d9e82

Generated ACI(s):
ubuntu-latest.aci
$ actool --debug validate ubuntu-latest.aci 
ubuntu-latest.aci: valid app container image
```

```
$ docker2aci docker://redis
Downloading 21e4345e9035: [====================================] 37.2 MB/37.2 MB
Downloading b3d362b23ec1: [====================================] 32 B/32 B
Downloading 29809ed33dfd: [====================================] 1.7 KB/1.7 KB
Downloading b720af9a6508: [====================================] 7.56 MB/7.56 MB
Downloading c215cb712b89: [====================================] 89.3 KB/89.3 KB
Downloading 8d9a45a71a91: [====================================] 611 KB/611 KB 
Downloading 130c4eb9410a: [====================================] 32 B/32 B
Downloading 1c255a1b1254: [====================================] 32 B/32 B
Downloading 7ebc2ece510e: [====================================] 32 B/32 B
Downloading 4454da7c7dbc: [====================================] 3.03 MB/3.03 MB
Downloading 3a8cd27bb3d5: [====================================] 95 B/95 B
Downloading d315f0a01142: [====================================] 32 B/32 B
Downloading e501d0146d1d: [====================================] 32 B/32 B
Downloading 40980abbab9f: [====================================] 196 B/196 B
Downloading 6755f61be70b: [====================================] 32 B/32 B
Downloading 54ca92b7c8d7: [====================================] 32 B/32 B
Downloading 06a1f75304ba: [====================================] 32 B/32 B

Converted volumes:
        name: "volume-data", path: "/data", readOnly: false

Converted ports:
        name: "6379-tcp", protocol: "tcp", port: 6379, count: 1, socketActivated: false

Generated ACI(s):
redis-latest.aci
$ actool --debug validate redis-latest.aci
redis-latest.aci: valid app container image
```

[aci]: https://github.com/appc/spec/blob/master/SPEC.md#app-container-image
[imageschema]: https://github.com/appc/spec/blob/master/spec/aci.md#image-manifest-schema
