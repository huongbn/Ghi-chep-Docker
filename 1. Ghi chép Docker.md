# Một vài tìm hiểu về Docker

## Mục lục

- [1. Docker netwok](#1)
    - [1.1 Overlay networks](#11)
    - [1.2 Disable netwoking for a container](#12)
- [2. Docker and iptables](#2)

<a name="1"></a>

# 1. Docker network

<a name="11"></a>

## 1.1 Overlay networks

- Khi tạo 1 swarm, sẽ có 2 networks mới được tạo ra:
    - ingress: Control and data traffic related to swarn service. Default swarm service connect to ingress
    - docker_gwbridge: Connect the individual Docker daemon to the other daemons in the swarm
- Lưu ý:
    - Chỉ swarm service mới có thể sử dụng overlay network, a standard container không thể sử dụng được
    - Muốn sử dụng được phải thêm `--attachable` flag 

### Encrypt traffic on an overlay network

- Sử dụng `--opt encrypted` khi tạo mới overlay network. No sẽ enabled IPSEC encryption 

### Customize the default ingress network

```
$ docker network create \
  --driver overlay \
  --ingress \
  --subnet=10.11.0.0/16 \
  --gateway=10.11.0.2 \
  --opt com.docker.network.driver.mtu=1200 \
  my-ingress
  ```

  ### Customize the docker_gwbridge interface

- Lưu ý: 
    - Muốn customize, phải thực hiện trước khi join host vào Swarm
    - Các bước thực hiện:
        - Stop Docker
        - Delete docker_gwbridge
            ```
            $ sudo ip link set docker_gwbridge down
            $ sudo ip link del dev docker_gwbridge
            ```
- Customize:
```
$ docker network create \
--subnet 10.11.0.0/16 \
--opt com.docker.network.bridge.name=docker_gwbridge \
--opt com.docker.network.bridge.enable_icc=false \
--opt com.docker.network.bridge.enable_ip_masquerade=true \
docker_gwbridge
```
[Tham khảo](https://docs.docker.com/engine/reference/commandline/network_create/#specify-advanced-options)

<a name="12"></a>

## 1.2 Disable networking for a container

Sử dụng `--network none` trong lúc run container
```
$ docker run --rm -dit \
  --network none \
  --name no-net-alpine \
  alpine:latest \
  ash
```
