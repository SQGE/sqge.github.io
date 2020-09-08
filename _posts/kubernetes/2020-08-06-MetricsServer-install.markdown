---
layout: post
title: "Metrics Server 安装"
subtitle: "Metrics Server 安装"
author: "duanzh"
header-img: "img/home-bg.jpg"
header-mask: 0.3
tags:
  - kubernetes
---

>本文介绍kubernetes1.18.8安装Metrics Server

#  #什么是metrics-server？

kubernetes 集群资源监控之前可以通过 heapster 来获取数据，在 1.11 开始开始逐渐废弃 heapster 了，采用 metrics-server 来代替，metrics-server 是集群的核心监控数据的聚合器，它从 kubelet 公开的 Summary API 中采集指标信息，metrics-server 是扩展的 APIServer，依赖于[kube-aggregator](https://github.com/kubernetes/kube-aggregator)，因为我们需要在 APIServer 中开启相关参数。

查看 APIServer 参数配置，确保你的 APIServer 启动参数中包含下的一些参数配置。
```
...
- --requestheader-client-ca-file=/etc/kubernetes/certs/proxy-ca.crt
- --proxy-client-cert-file=/etc/kubernetes/certs/proxy.crt
- --proxy-client-key-file=/etc/kubernetes/certs/proxy.key
- --requestheader-allowed-names=aggregator
- --requestheader-extra-headers-prefix=X-Remote-Extra-
- --requestheader-group-headers=X-Remote-Group
- --requestheader-username-headers=X-Remote-User
- --enable-aggregator-routing=true
...
```
>如果您未在 master 节点上运行 kube-proxy，则必须确保 kube-apiserver 启动参数中包含--enable-aggregator-routing=true

# #安装
从[Metrics Server版本](https://github.com/kubernetes-sigs/metrics-server/releases)下载最新版本，并按如下所示进行部署：
```
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/download/v0.3.7/components.yaml
```
>目前我采用最新版本部署，v0.3.7

# #完整的资源清单文件
```
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: system:aggregated-metrics-reader
  labels:
    rbac.authorization.k8s.io/aggregate-to-view: "true"
    rbac.authorization.k8s.io/aggregate-to-edit: "true"
    rbac.authorization.k8s.io/aggregate-to-admin: "true"
rules:
- apiGroups: ["metrics.k8s.io"]
  resources: ["pods", "nodes"]
  verbs: ["get", "list", "watch"]
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: metrics-server:system:auth-delegator
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: system:auth-delegator
subjects:
- kind: ServiceAccount
  name: metrics-server
  namespace: kube-system
---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: metrics-server-auth-reader
  namespace: kube-system
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: extension-apiserver-authentication-reader
subjects:
- kind: ServiceAccount
  name: metrics-server
  namespace: kube-system
---
apiVersion: apiregistration.k8s.io/v1beta1
kind: APIService
metadata:
  name: v1beta1.metrics.k8s.io
spec:
  service:
    name: metrics-server
    namespace: kube-system
  group: metrics.k8s.io
  version: v1beta1
  insecureSkipTLSVerify: true
  groupPriorityMinimum: 100
  versionPriority: 100
---
apiVersion: v1
kind: ServiceAccount
metadata:
  name: metrics-server
  namespace: kube-system
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: metrics-server
  namespace: kube-system
  labels:
    k8s-app: metrics-server
spec:
  selector:
    matchLabels:
      k8s-app: metrics-server
  template:
    metadata:
      name: metrics-server
      labels:
        k8s-app: metrics-server
    spec:
      serviceAccountName: metrics-server
      volumes:
      # mount in tmp so we can safely use from-scratch images and/or read-only containers
      - name: tmp-dir
        emptyDir: {}
      containers:
      - name: metrics-server
        image: coll/metrics-server:v0.3.7   # image默认是谷歌，修改为可用的
        imagePullPolicy: IfNotPresent
        args:
          - --cert-dir=/tmp
          - --secure-port=4443
        command:   #新增command配置
        - /metrics-server
        - --kubelet-insecure-tls 
        - --kubelet-preferred-address-types=InternalIP
        ports:
        - name: main-port
          containerPort: 4443
          protocol: TCP
        securityContext:
          readOnlyRootFilesystem: true
          runAsNonRoot: true
          runAsUser: 1000
        volumeMounts:
        - name: tmp-dir
          mountPath: /tmp
      nodeSelector:
        kubernetes.io/os: linux
---
apiVersion: v1
kind: Service
metadata:
  name: metrics-server
  namespace: kube-system
  labels:
    kubernetes.io/name: "Metrics-server"
    kubernetes.io/cluster-service: "true"
spec:
  selector:
    k8s-app: metrics-server
  ports:
  - port: 443
    protocol: TCP
    targetPort: main-port
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: system:metrics-server
rules:
- apiGroups:
  - ""
  resources:
  - pods
  - nodes
  - nodes/stats
  - namespaces
  - configmaps
  verbs:
  - get
  - list
  - watch
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: system:metrics-server
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: system:metrics-server
subjects:
- kind: ServiceAccount
  name: metrics-server
  namespace: kube-system
```
# #遇到的问题
```
E0908 03:43:06.048002       1 reststorage.go:135] unable to fetch node metrics for node "build-02": no metrics known for node
E0908 03:43:06.048022       1 reststorage.go:135] unable to fetch node metrics for node "build-01": no metrics known for node
E0908 03:43:06.048030       1 reststorage.go:135] unable to fetch node metrics for node "build-03": no metrics known for node
E0908 03:43:48.828393       1 manager.go:111] unable to fully collect metrics: [unable to fully scrape metrics from source kubelet_summary:build-03: unable to fetch metrics from Kubelet build-03 (build-03): Get https://build-03:10250/stats/summary?only_cpu_and_memory=true: dial tcp: lookup build-03 on 10.96.0.10:53: server misbehaving, unable to fully scrape metrics from source kubelet_summary:build-01: unable to fetch metrics from Kubelet build-01 (build-01): Get https://build-01:10250/stats/summary?only_cpu_and_memory=true: dial tcp: lookup build-01 on 10.96.0.10:53: server misbehaving, unable to fully scrape metrics from source kubelet_summary:build-02: unable to fetch metrics from Kubelet build-02 (build-02): Get https://build-02:10250/stats/summary?only_cpu_and_memory=true: dial tcp: lookup build-02 on 10.96.0.10:53: server misbehaving]
```
因为部署集群的时候，CA 证书并没有把各个节点的 IP 签上去，所以这里 metrics-server 通过 IP 去请求时，提示签的证书没有对应的 IP（错误：x509: cannot validate certificate for 192.168.33.11 because it doesn’t contain any IP SANs），我们可以添加一个--kubelet-insecure-tls参数跳过证书校验：
```
        command:   #新增command配置
        - /metrics-server
        - --kubelet-insecure-tls 
        - --kubelet-preferred-address-types=InternalIP
```

添加上面以上参数，然后再重新安装。
```
kubectl apply -f components.yaml
```
过一会可以看到，使用Metrics-Server收集到节点信息，说明Metrics-Server安装成功。
```
[root@build-01 manifests]# kubectl top node
NAME       CPU(cores)   CPU%   MEMORY(bytes)   MEMORY%   
build-01   142m         3%     3818Mi          50%       
build-02   83m          2%     3674Mi          48%       
build-03   2782m        69%    4262Mi          55%   
```

#### 相关文档：
https://github.com/kubernetes-sigs/metrics-server
https://www.qikqiak.com/post/install-metrics-server/

##### 转载请注明出处，本文采用 CC4.0 协议授权
