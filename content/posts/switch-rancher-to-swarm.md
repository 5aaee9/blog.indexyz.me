---
title:  将 Docker 集群管理转为了 swarm
date: '2018-10-01T11:03:00+08:00'
categories:
- Docker
---
之前使用 rancher 做为我个人的 docker 容器管理引擎, rancher 在大部分时候都工作的很好. 但是 Rancher 1 将会在最早于明年夏天 ([Rancher Forums][1]) 结束支持, 因为 k8s 的占用过于庞大, 我因此不是很想转换到 rancher 2 上 (而且也不能平滑升级) 这时候就尝试试一下 swarm

~~swarm 真香~~

<!--more-->


# 节点的部署
在 manager 机器上使用 `docker swarm init` 你将会得到用与加入 swarm 集群的命令 将这个命令在 worker 机器上执行就可以将机器加入集群了
## 注意事项
### 在 1:1 NAT 的机器上网络通信的问题
在某些 1:1 NAT 的机器上 (例如 Google Cloud / AWS) 上你可能会遇到这个问题, 问题具体表现为你加入集群之后是没问题的 但是节点之间 overlay 的 network 并不能跨机通信

答案: 在 docker swarm join 的时候加上 `--advertise-addr 外网IP` 这个 参数

# 创建服务
## traefik
[Traefik][2] 是一个用使用 golang 编写的负载均衡 / 反向代理服务端, 可以通过 service 的 label 来配置外网的访问

首先创建文件夹
```bash
mkdir -p /data/traefik
touch acme.json  traefik.toml
```

然后编辑 `/data/traefik/traefik.toml` 这一个配置文件 以下为我的配置文件
```plain
debug = true
defaultEntryPoints = ["http", "https"]

[web]
address = ":8080"
ReadOnly = true
  [web.auth.basic]
  users = ["Indexyz:$apr1$h7AhHWe1$I7OTEhwmTR5GUg1mPv6yS/"]

[entryPoints]
  [entryPoints.http]
  address = ":80"
    [entryPoints.http.redirect]
    entryPoint = "https"
  [entryPoints.https]
  address = ":443"
  compress = true
    [entryPoints.https.tls]

[retry]

[acme]
email = "acme@indexyz.me"
storage = "acme.json"
entryPoint = "https"
onHostRule = true
[acme.httpChallenge]
  entryPoint = "http"
```

使用以下命令来创建 traefik 服务
```bash
docker network create --driver overlay \
    --attachable \
    traefik

docker service create --name traefik \
    --name traefik \
    --constraint=node.role==manager \
    --publish 80:80 \
    --publish 443:443 \
    --mount type=bind,source=/var/run/docker.sock,target=/var/run/docker.sock \
    --mount type=bind,source=/data/traefik/traefik.toml,target=/traefik.toml \
    --mount type=bind,source=/data/traefik/acme.json,target=/acme.json \
    --network traefik \
    --label traefik.port=8080 \
    --label traefik.frontend.rule=Host:traefik.indexyz.me \
    --label traefik.docker.network=traefik \
    --label 'traefik.frontend.auth.basic=Indexyz:$PASSWORD$' \
    traefik \
    --docker \
    --docker.swarmmode \
    --docker.domain=indexyz.me \
    --docker.watch \
    --web

```
将其中的 `$PASSWORD$` 替换为 apr1 加密过的密码 (使用 `openssl passwd --apr1`)

### docker-proxy
一些事件只能在 master 的 docker.sock 上监听到 因此很多应用都有部署到 manage 上的要求 我们可以使用 `rancher/socat-docker` 来将 master 的 docker.sock 在 tcp 上监听

利用以下命令来运行 docker-proxy
```bash
docker network create --driver overlay \
    --attachable \
    docker-proxy

docker service create \
    --name docker-proxy \
    --mount "type=bind,source=/var/run/docker.sock,target=/var/run/docker.sock" \
    --constraint 'node.role==manager' \
    --network docker-proxy \
    rancher/socat-docker
```

### Portainer
[Portainer][3] 是一个 可视化的 docker 管理器, 支持 swarm 集群的管理, 毕竟我们一直用命令行进行创建 service 比较麻烦

使用以下命令来创建 Portainer 服务
```bash
docker service create \
    --name portainer \
    --replicas=1 \
    --constraint 'node.role == manager' \
    --mount type=bind,src=/var/run/docker.sock,dst=/var/run/docker.sock \
    --mount type=bind,src=/data/portainer,dst=/data \
    --label traefik.port=9000 \
    --label traefik.frontend.rule=Host:portainer.indexyz.me \
    --label traefik.docker.network=traefik \
    --network traefik \
    portainer/portainer \
    -H unix:///var/run/docker.sock
```


  [1]: https://forums.rancher.com/t/future-support-for-rancher-1-x-cattle/7442
  [2]: https://traefik.io/
  [3]: https://portainer.io/
