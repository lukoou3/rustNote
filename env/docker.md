
## docker环境运行rust
rust build的程序默认会有一些动态链接，在别的机器上可能不能运行，所以使用docker运行。

目标尽量使用小的镜像直接运行编译好的文件，官方的rust镜像还是比较大的，1G多，我们使用rust镜像编译程序，然后使用小的镜像运行程序，这里使用debian:bookworm-slim。

### 使用rust镜像编译程序

```
docker run -it --name rust rust:1.85 bash

docker cp Cargo.toml rust:/opt
docker cp src rust:/opt

apt update
apt install -y cmake
cargo build --release

docker cp rust:/opt/target/release/retl  ./
```

### 使用其他的镜像运行

rust:1.85-slim可以直接运行，不过他有800M，有点大
```
docker run -it --name rustslim rust:1.85-slim bash

docker cp retl rustslim:/opt
docker cp config/ rustslim:/opt


root@cc0095cde5bf:/opt# ./retl config/application_test.yaml
```

debian:bookworm-slim比较小，安装个别依赖就行，不过debian:bookworm-slim中没有top, vi等工具。

```
docker run -it --name debian debian:bookworm-slim bash

docker cp retl debian:/opt
docker cp config/ debian:/opt

apt-get update
apt-get install -y libssl3

/opt# ldd retl
linux-vdso.so.1 (0x00007fff701ec000)
libssl.so.3 => /lib/x86_64-linux-gnu/libssl.so.3 (0x00007ffac735f000)
libcrypto.so.3 => /lib/x86_64-linux-gnu/libcrypto.so.3 (0x00007ffac6ed9000)
libz.so.1 => /lib/x86_64-linux-gnu/libz.so.1 (0x00007ffac6eba000)
libgcc_s.so.1 => /lib/x86_64-linux-gnu/libgcc_s.so.1 (0x00007ffac6e9a000)
libm.so.6 => /lib/x86_64-linux-gnu/libm.so.6 (0x00007ffac6dbb000)
libc.so.6 => /lib/x86_64-linux-gnu/libc.so.6 (0x00007ffac6bd8000)
/lib64/ld-linux-x86-64.so.2 (0x00007ffac811a000)


./retl
```

### 制作运行环境镜像
Dockerfile
```
FROM debian:bookworm-slim

RUN apt-get update && apt-get install -y libssl3 && \
  mkdir /opt/app && \
  rm -rf /var/lib/apt/lists/* && rm -rf /var/cache/apk/*


```

build.sh
```
#!/bin/bash
echo "开始构建镜像"
images_tag=1.0.0

docker rmi -f debianrs:$images_tag
rm -rf debianrs-${images_tag}.tar

docker build -t="debianrs:${images_tag}" -f ./Dockerfile .

echo "docker save debianrs:${images_tag} -o debianrs-${images_tag}.tar"
#docker save debianrs:${images_tag} -o debianrs-${images_tag}.tar

```


```
debianrs]# docker images | grep debian
debianrs                                                              1.0.0                                          123914b5b08c        About a minute ago   81.6MB
debian                                                                bookworm-slim                                  65bf02b05d2e        2 weeks ago          74.8MB
```

导出镜像：
```
docker save debianrs:1.0.0 -o debianrs-1.0.0.tar

docker load -i debianrs-1.0.0.tar
```

```
docker run -d --name debianrs --net host \
-v /home/lfc/rs/app:/opt/app \
debianrs:1.0.0 tail -f /dev/null

docker exec -it debianrs bash

docker stop debianrs
docker rm debianrs
docker rm -f debianrs

docker run -it --name debianrs -v /home/lfc/rs/app:/opt/app debianrs:1.0.0 bash


docker run -it --name debianrs -v /home/lfc/docker-images/debianrs/app:/opt/app debianrs:1.0.0 bash
```

### 编译与运行

编译：

docker-compose.yml
```yml
version: '2'
services:
  cmak:
    image: rust:1.85
    container_name: rust
    volumes:
      - ./app:/opt/app
    network_mode: "host"
    command: ["sleep", "infinity"]

```

运行：

docker-compose.yml
```yml
version: '2'
services:
  cmak:
    image: debianrs:1.0.0
    container_name: debianrs
    volumes:
      - ./app:/opt/app
    network_mode: "host"
    command: ["sleep", "infinity"]

```


