# Kubernetes + Ubuntu 18.04 漏洞环境搭建

## 前置知识

本 Kubernetes + Ubuntu 18.04 漏洞环境适用于 [Awesome-POC](https://github.com/Threekiii/Awesome-POC/tree/master/%E4%BA%91%E5%AE%89%E5%85%A8%E6%BC%8F%E6%B4%9E) 中与 Kubernetes 相关的部分云安全漏洞。

各组件版本如下：

```
Docker version: 18.09.3/19.03.6
minikube version: v1.35.0
Kubectl Client Version: v1.32.3
Kubectl Server Version: v1.32.0
```

本环境可用于复现以下漏洞：

| 类别   | 漏洞名称                                                      | CDK(v1.5.5) Exploit                                                                                                                                                                               | 文档链接                                                                   |
| ---- | --------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------- |
| 容器逃逸 | 挂载 docker.sock 导致容器逃逸                                     | <p><a href="https://github.com/Xyntax/CDK/wiki/Exploit:-docker-sock-check">docker-sock-check</a><br><a href="https://github.com/Xyntax/CDK/wiki/Exploit:-docker-sock-pwn">docker-sock-pwn</a></p> | [link](<挂载 docker.sock 导致容器逃逸.md>)                                     |
| 容器逃逸 | 挂载 log 目录导致容器逃逸                                           | -                                                                                                                                                                                                 | [link](<挂载 log 目录导致容器逃逸.md>)                                           |
| 容器逃逸 | 挂载宿主机 procfs 系统导致容器逃逸                                     | [mount-procfs](https://github.com/Xyntax/CDK/wiki/Exploit:-mount-procfs)                                                                                                                          | [link](<挂载宿主机 procfs 系统导致容器逃逸.md>)                                     |
| 容器逃逸 | Containerd 漏洞导致容器逃逸 CVE-2020-15257                        | [shim-pwn](https://github.com/Xyntax/CDK/wiki/Exploit:-shim-pwn)                                                                                                                                  | [link](<Containerd 漏洞导致容器逃逸 CVE-2020-15257.md>)                        |
| 容器逃逸 | Docker copy 漏洞导致容器逃逸 CVE-2019-14271                       | [docker-api-pwn](https://github.com/Xyntax/CDK/wiki/Exploit:-docker-api-pwn)                                                                                                                      | [link](<Docker copy 漏洞导致容器逃逸 CVE-2019-14271.md>)                       |
| 容器逃逸 | 挂载重写 cgroup devices.allow 导致容器逃逸                          | [rewrite-cgroup-devices](https://github.com/cdk-team/CDK/wiki/Exploit:-rewrite-cgroup-devices)                                                                                                    | [link](<挂载重写 cgroup devices.allow 导致容器逃逸.md>)                          |
| 容器逃逸 | Linux 内核 cgroups v1 逻辑错误导致容器逃逸 CVE-2022-0492              | [mount-cgroup](https://github.com/Xyntax/CDK/wiki/Exploit:-mount-cgroup)                                                                                                                          | [link](<Linux 内核 cgroup v1 逻辑错误导致容器逃逸 CVE-2022-0492.md>)               |
| 容器逃逸 | Kubernetes privileged 特权容器导致容器逃逸                          | [mount-disk](https://github.com/Xyntax/CDK/wiki/Exploit:-mount-disk)                                                                                                                              | [link](<Kubernetes privileged 特权容器导致容器逃逸.md>)                          |
| 持久化  | Kubernetes 部署 Shadow API Server                           | [k8s-shadow-apiserver](https://github.com/cdk-team/CDK/wiki/Exploit:-k8s-shadow-apiserver)                                                                                                        | [link](<Kubernetes 部署 Shadow API Server.md>)                           |
| 持久化  | Kubernetes 部署后门 CronJob                                   | [k8s-cronjob](https://github.com/cdk-team/CDK/wiki/Exploit:-k8s-cronjob)                                                                                                                          | [link](<Kubernetes 部署后门 CronJob.md>)                                   |
| 持久化  | Kubernetes 部署后门 Daemonset                                 | [k8s-backdoor-daemonset](https://github.com/cdk-team/CDK/wiki/Exploit:-k8s-backdoor-daemonset)                                                                                                    | [link](<Kubernetes 部署后门 Daemonset.md>)                                 |
| 权限提升 | Kubernetes 利用 nodes proxy 子资源进行权限提升                       | -                                                                                                                                                                                                 | [link](<Kubernetes 利用 nodes proxy 子资源进行权限提升.md>)                       |
| 命令执行 | Docker build 漏洞导致命令执行 CVE-2019-13139                      | -                                                                                                                                                                                                 | [link](<Docker build 漏洞导致命令执行 CVE-2019-13139.md>)                      |
| 命令执行 | Docker daemon api 未授权访问漏洞 RCE                             | -                                                                                                                                                                                                 | [link](<Docker daemon api 未授权访问漏洞 RCE.md>)                             |
| 命令执行 | Kubernetes Ingress-nginx admission 远程代码执行漏洞 CVE-2025-1974 | -                                                                                                                                                                                                 | [link](<Kubernetes Ingress-nginx admission 远程代码执行漏洞 CVE-2025-1974.md>) |
| 命令执行 | Kubernetes API Server 未授权命令执行                             | -                                                                                                                                                                                                 | [link](<Kubernetes API Server 未授权命令执行.md>)                             |
| 信息窃取 | Kubernetes etcd 未授权访问                                     | -                                                                                                                                                                                                 | [link](<Kubernetes etcd 未授权访问.md>)                                     |

## 环境搭建

### Docker 18.09.3

> 注意，此版本是一个较老的版本（发布于 2019 年 3 月），它存在多个已知的安全漏洞。**安装脚本仅限于漏洞环境，请勿用于生产环境。**

安装脚本 `install_docker_18.09.03.sh`：

```
#!/bin/bash

set -e

echo "🔧 卸载旧版本 Docker（如果有）..."
sudo apt remove -y docker docker-engine docker.io containerd runc || true

echo "🔓 解除版本锁定 (如果有)..."
sudo apt-mark unhold docker-ce docker-ce-cli containerd.io || true

echo "🧹 删除错误的 Docker 源..."
sudo rm -f /etc/apt/sources.list.d/docker.list || true
sudo sed -i '/download.docker.com/d' /etc/apt/sources.list

echo "🌐 添加清华大学 Docker 镜像源 GPG key..."
wget -qO - https://mirrors.tuna.tsinghua.edu.cn/docker-ce/linux/ubuntu/gpg | sudo apt-key add -

echo "📚 添加清华大学 Docker 镜像源..."
echo "deb [arch=amd64] https://mirrors.tuna.tsinghua.edu.cn/docker-ce/linux/ubuntu bionic stable" \
  | sudo tee /etc/apt/sources.list.d/docker.list

echo "📦 更新软件包索引..."
sudo apt update

echo "🔍 查找 Docker 18.09.3..."
VERSION_STRING=$(apt-cache madison docker-ce | grep 18.09.3 | head -n1 | awk '{print $3}')
if [ -z "$VERSION_STRING" ]; then
  echo "❌ 找不到 Docker 18.09.3"
  exit 1
fi
echo "✅ 找到版本：$VERSION_STRING"

echo "⬇️ 安装 Docker 版本 $VERSION_STRING ..."
sudo apt install -y docker-ce=$VERSION_STRING docker-ce-cli=$VERSION_STRING containerd.io

echo "📌 锁定版本，防止自动升级..."
sudo apt-mark hold docker-ce docker-ce-cli containerd.io

echo "✅ 安装完成，当前版本："
docker --version
```

> 其他版本 Docker 修改 `18.09.3` 版本号即可。

### Kubectl v1.32.3

安装最新版本：

```
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
```

或直接安装 v1.32.3：

```
curl -LO "https://dl.k8s.io/release/v1.32.3/bin/linux/amd64/kubectl"
```

将下载的 `kubectl` 安装到系统路径 `/usr/local/bin/`：

```
sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl 
```

### Minikube v1.35.0

安装最新版本：

```
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
```

将下载的 `minikube` 安装到系统路径 `/usr/local/bin/`：

```
sudo install minikube-linux-amd64 /usr/local/bin/minikube
```

## 环境启动

### 启动 Minikube 集群

```
minikube start --force
```

由于网络原因，国内可以拉取 `kicbase`，自定义基础镜像源启动 minikube：

```
docker pull registry.cn-hangzhou.aliyuncs.com/google_containers/kicbase:v0.0.46
minikube start --force --base-image='registry.cn-hangzhou.aliyuncs.com/google_containers/kicbase:v0.0.46'
```

### 查看 Minikube 集群状态

```
minikube status
```

![](../.gitbook/assets/image-20250422113722290.png)

### 查看集群节点状态

```
kubectl get nodes
```

![](../.gitbook/assets/image-20250422113744552.png)

### 查看所有 Pod 状态

```
kubectl get pods --all-namespaces
```

![](../.gitbook/assets/image-20250422113927638.png)

### 停止 Minikube 集群

```
minikube stop
```

### 删除 Minikube 集群

```
minikube delete
```
