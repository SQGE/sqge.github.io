---
layout: post
title: "kubernetes如何支持从私有仓库拉取镜像"
subtitle: "kubernetes如何支持从私有仓库拉取镜像"
author: "duanzh"
header-img: "img/home-bg.jpg"
header-mask: 0.3
tags:
  - kubernetes
---

>本文介绍如何使用 Secret 从私有的 Docker 镜像仓库或代码仓库拉取镜像来创建 Pod。

# #在集群中创建保存授权令牌的 Secret

```
kubectl create secret docker-registry regsecret \
  --docker-server=registry-internal.cn-hangzhou.aliyuncs.com \
  --docker-username=USERNAME \
  --docker-password=PASSWORD \
  --docker-email=YOUR-EMAIL-ADDR
```
其中：

*   **regsecret：** 指定密钥的键名称，可自行定义。
*   **--docker-server：**指定 Docker 仓库地址。
*   **--docker-username**: 指定 Docker 仓库用户名。
*   **--docker-password：**指定 Docker 仓库登录密码。
*   **--docker-email：**指定邮件地址（选填）。

# #创建一个使用你的 Secret 的 Pod
```
apiVersion: v1
kind: Pod
metadata:
  name: private-reg
spec:
  containers:
  - name: private-reg-container
    image: <your-private-image>
  imagePullSecrets:
  - name: regsecret 
```
其中：

*   `imagePullSecrets` 是声明拉取镜像时需要指定密钥。
*   `regsecret` 必须和上面生成密钥的键名一致。
*   `image` 中的 Docker 仓库名称必须和 `--docker-server` 中的 Docker 仓库名一致。

详情信息参见官方文档 [使用私有仓库](https://kubernetes.io/docs/concepts/containers/images/#using-a-private-registry)。




##### 转载请注明出处，本文采用 CC4.0 协议授权
