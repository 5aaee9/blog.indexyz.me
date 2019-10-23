---
title: k3s 踩坑
date: '2019-04-16T11:44:00+08:00'
updated: 2019-10-23T07:45:12.143Z
tags:
  - Docker
  - Kubernetes
  - Linux
categories:
  - Kubernetes
---
Kubernetes 之前很火, 但是因为对每个计算节点的配置要求非常之高, 我一直是在使用 Rancher 的 `Cattle` 而并非使用 k8s.
最近听说 Rancher Labs 推出了 k3s 并且据说资源占用特别小, 那就来踩下坑（跑

<!--more-->


# 开始安装
## 已知问题
因为默认的 Ingress `traefik` 有问题, 在 Rancher 2 上添加 Load Balancing 会导致这个 Ingress 一直 `Initializing`, 但是实际上是能用的.

[Issue in Rancher #19135][1]

同时因为 `traefik` 很多奇怪的问题, 我将会将其换为 `nginx-ingress`

因为我是要跨平台组集群 而 `flannel` 在各个节点之间的通信是不加密的, 此处不使用 `flannel` 作为 cni 驱动, 我将其换为了 `calico` 和 `zerotier` 的组合.

> 此处主控和节点已经处在同一个 zerotier 网络中

## 安装主控端
```bash
curl -sfL https://get.k3s.io -o k3s.sh
chmod +x k3s.sh

bash k3s.sh --no-deploy traefik --no-flannel

rm -f k3s.sh
```
## 修改主控端 node ip
如果你和我一样是使用的 zerotier 或者 wireguard 等软件搭建的内网, 需要修改 server 和 agent 的启动参数

在运行安装脚本的时候加入 `--node-ip 内网IP --flannel-iface 网卡`

## 安装 calico
> Calico 也有一个坑, 如果你使用了 dind (docker in docker), dockerd 默认建立出来的网卡 MTU 是 1500, 但是 calico 的网卡 MTU 只有 `1450` 会导致容器无法上网
> 解决方法就是修改 calico 的 configmap 中 mtu 相关的设定 然后再 `kubectl delete pod $(kubectl get pod -n kube-system | grep calico-node | awk '{print $1}') -n kube-system`

```bash
curl https://docs.projectcalico.org/v3.6/getting-started/kubernetes/installation/hosted/kubernetes-datastore/calico-networking/1.7/calico.yaml -O
sed -i -e "s?192.168.0.0/16?10.42.0.0/16?g" calico.yaml
kubectl apply -f calico.yaml
```

## 安装被控端
```bash
curl -sfL https://get.k3s.io -o k3s.sh
chmod +x k3s.sh

K3S_URL=https://SERVER_IP:6443 \
    K3S_TOKEN=TOKEN \
    bash k3s.sh --no-flannel

rm -f k3s.sh
```

> Token 可以从主控端的 `/var/lib/rancher/k3s/server/node-token` 中获得

# 使用

## 安装 Nginx-Ingress
apply 这个 yml
```yaml
apiVersion: helm.cattle.io/v1
kind: HelmChart
metadata:
  name: nginx-ingress
  namespace: kube-system
spec:
  chart: stable/nginx-ingress
  valuesContent: |-
    controller:
      kind: DaemonSet
      daemonset:
        useHostPort: true
      nodeSelector:
        nodeRole: nginx-edge
      service:
        type: ClusterIP
```
> 注意要给能 nginx 访问的 node 打上 `nodeRole=nginx-edge` 的 label

## 安装 cert-manager
> 此处例子为配置 DNS 解析, 使用 DigitalOcean 更多操作请查看 [官方文档][2]

因为 k3s 的问题, 我们需要部署不带 webhook 的 cert-manager

[Issue in cert-manager #1519][3]
[Issue in k3s #117][4]
[Issue in k3s #120][5]

```bash
kubectl create namespace cert-manager
kubectl label namespace cert-manager certmanager.k8s.io/disable-validation=true

kubectl apply -f https://raw.githubusercontent.com/jetstack/cert-manager/release-0.7/deploy/manifests/cert-manager-no-webhook.yaml
```
然后这时候 cert-manager 就安装完了
新建一个 Screct 存一下 DO 的 API Key
```bash
echo "TOKEN" > token
kubectl create secret generic digitalocean-dns-api --namespace=cert-manager --from-file=token
```
然后创建一个 Issuer
```yml
apiVersion: certmanager.k8s.io/v1alpha1
kind: ClusterIssuer
metadata:
  name: digitalocean-issuer
  namespace: kube-system
spec:
  acme:
    email: indexyz@protonmail.com
    server: https://acme-v02.api.letsencrypt.org/directory
    privateKeySecretRef:
      name: acme-issuer-account-key
    dns01:
      providers:
      - name: digitalocean-acme
        digitalocean:
          tokenSecretRef:
            name: digitalocean-dns-api
            key: token
```
然后我们就能签下来我们的证书了
```yml
apiVersion: certmanager.k8s.io/v1alpha1
kind: Certificate
metadata:
  name: dashboard-cert
  namespace: kube-system
spec:
  secretName: dashboard-cert-tls
  issuerRef:
    name: digitalocean-issuer
    kind: ClusterIssuer
  commonName: 'kubernetes.bogon.dev'
  dnsNames:
  - kubernetes.bogon.dev
  acme:
    config:
    - dns01:
        provider: digitalocean-acme
      domains:
      - kubernetes.bogon.dev
```

## 安装 kubernetes dashboard
```bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v1.10.1/src/deploy/recommended/kubernetes-dashboard.yaml
```
然后我们需要建立一个能够 Cluster Role view 的账户
```yml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: kubedash
  namespace: kube-system
---
kind: ClusterRoleBinding
apiVersion: rbac.authorization.k8s.io/v1beta1
metadata:
  name: dashboard
subjects:
  - kind: ServiceAccount
    name: kubedash
    namespace: kube-system
roleRef:
  kind: ClusterRole
  name: cluster-admin
  apiGroup: rbac.authorization.k8s.io
```
然后使用 `kubectl apply -f` 进行导入并使用
```bash
kubectl -n kube-system describe secret $(kubectl -n kube-system get secret | grep kubedash | awk '{print $1}')
```
获取到账户的 Token

为了能进行外网访问， 我们还需要建立一个 Ingress 来将其开放到公网

创建一个 TLS 证书类型的 secret

> 如果你使用 cert-manager 创建证书的话, 可以跳过此步

```bash
kubectl create secret tls dashboard-cert-tls --key k8s.key --cert k8s.crt --namespace=kube-system
# 证书哪里拿我应该不用讲了
```

创建 Ingress
```yml
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: dashboard-ingress
  namespace: kube-system
  annotations:
    ingress.kubernetes.io/protocol: https
    nginx.ingress.kubernetes.io/backend-protocol: https

spec:
  rules:
  - host: kubernetes.bogon.dev
    http:
      paths:
        - backend:
            serviceName: kubernetes-dashboard
            servicePort: 443
  tls:
  - hosts:
    - kubernetes.bogon.dev
    secretName: dashboard-cert-tls
```

然后我们就完成了最简单的一部分
![k8s dashboard](https://i.loli.net/2019/04/10/5cadecf754402.png)

## ChangeLog
- 2019/8/14: 添加了 修改主控端 node ip 部分, 修改了 HelmCharts 的 apiVersion, 对应 [Release v0.6.1][6]


  [1]: https://github.com/rancher/rancher/issues/19135
  [2]: https://docs.cert-manager.io/en/latest/
  [3]: https://github.com/jetstack/cert-manager/issues/1519
  [4]: https://github.com/rancher/k3s/issues/117
  [5]: https://github.com/rancher/k3s/issues/120
  [6]: https://github.com/rancher/k3s/releases/tag/v0.6.1
