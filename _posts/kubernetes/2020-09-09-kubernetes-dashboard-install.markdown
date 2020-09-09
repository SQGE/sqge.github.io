---
layout: post
title: "kubernetes1.18.8 安装dashboard"
subtitle: "kubernetes1.18.8 安装dashboard"
author: "duanzh"
header-img: "img/home-bg.jpg"
header-mask: 0.3
tags:
  - kubernetes
---

>本章介绍 kubernetes1.18.8 安装dashboard

Kubernetes仪表板是Kubernetes集群的基于Web的通用UI。它允许用户管理集群中运行的应用程序并对其进行故障排除，以及管理集群本身。

## ## 安装
```
kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.0.4/aio/deploy/recommended.yaml
```
>如果不能执行，可以去[dashboard](https://github.com/kubernetes/dashboard)官方项目搜索`recommended`关键字，下载到本地进行执行

## ## 修改servcie (NodePort方式）
```
kind: Service
apiVersion: v1
metadata:
  labels:
    k8s-app: kubernetes-dashboard
  name: kubernetes-dashboard
  namespace: kubernetes-dashboard
spec:
  type: NodePort # NodePort访问
  ports:
    - port: 443
      targetPort: 8443
      nodePort: 31000  # 固定端口
  selector:
    k8s-app: kubernetes-dashboard
```
## ## 获取 dashboard 的资源对象
```
kubectl get all -n kubernetes-dashboard
```

## ## 创建身份验证令牌（RBAC）
```
# 创建一个新的 ServiceAccount
kubectl apply -f - <<EOF
apiVersion: v1
kind: ServiceAccount
metadata:
  name: admin-user
  namespace: kubernetes-dashboard
EOF
# 将上面的 SA 绑定到系统的 cluster-admin 这个集群角色上
kubectl apply -f - <<EOF
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: admin-user
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
- kind: ServiceAccount
  name: admin-user
  namespace: kubernetes-dashboard
EOF
```
## ## 获取到的 Token 数据 
```
kubectl -n kubernetes-dashboard describe secret $(kubectl -n kubernetes-dashboard get secret | grep admin-user | awk '{print $1}')
```

## ## 访问dashboard 
```
http://NodeIP:31000
```

## #相关文档
https://www.qikqiak.com/post/deploy-k8s-on-win-use-wsl2/


##### 转载请注明出处，本文采用 CC4.0 协议授权
