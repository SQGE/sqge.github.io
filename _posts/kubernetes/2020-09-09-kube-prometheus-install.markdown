---
layout: post
title: "kubernetes1.18.8 安装kube-prometheus"
subtitle: "kubernetes1.18.8 安装kube-prometheus"
author: "duanzh"
header-img: "img/home-bg.jpg"
header-mask: 0.3
tags:
  - kubernetes
---

>本章介绍 kubernetes1.18.8 安装kube-prometheus

kube-prometheus 是一整套监控解决方案，它使用 Prometheus 采集集群指标，Grafana 做展示，包含如下组件：

- The Prometheus Operator
- Highly available Prometheus
- Highly available Alertmanager
- Prometheus node-exporter
- Prometheus Adapter for Kubernetes Metrics APIs （k8s-prometheus-adapter）
- kube-state-metrics
- Grafana

其中 `k8s-prometheus-adapter `使用 Prometheus 实现了 `metrics.k8s.io `和 `custom.metrics.k8s.io API`，所以不需要再部署 `metrics-server`。 

## #下载
```
https://github.com/prometheus-operator/kube-prometheus.git
```


## #修改service类型为NodePort
prometheus service
```
$ cat prometheus-service.yaml
apiVersion: v1
kind: Service
metadata:
  labels:
    prometheus: k8s
  name: prometheus-k8s
  namespace: monitoring
spec:
  type: NodePort
  ports:
  - name: web
    port: 9090
    targetPort: web
    nodePort: 30090
  selector:
    app: prometheus
    prometheus: k8s
  sessionAffinity: ClientIP
```
alertmanager service
```
$ cat alertmanager-service.yaml 
apiVersion: v1
kind: Service
metadata:
  labels:
    alertmanager: main
  name: alertmanager-main
  namespace: monitoring
spec:
  type: NodePort
  ports:
  - name: web
    port: 9093
    targetPort: web
    nodePort: 30093
  selector:
    alertmanager: main
    app: alertmanager
  sessionAffinity: ClientIP
```
grafana service
```
$ cat grafana-service.yaml
apiVersion: v1
kind: Service
metadata:
  labels:
    app: grafana
  name: grafana
  namespace: monitoring
spec:
  type: NodePort
  ports:
  - name: http
    port: 3000
    targetPort: http
    nodePort: 32000
  selector:
    app: grafana
```

## #安装
```
cd kube-prometheus/
kubectl apply -f manifests/setup # 安装 prometheus-operator
kubectl apply -f manifests/      # 安装 promethes metric adapter
```

## #访问地址
```
http://NodeIP:32000/           # grafana
http://NodeIP:30090/           # prometheus
http://NodeIP:30093/           # alertmanager 
``` 


## #相关文档
https://blog.csdn.net/guoxiaobo2010/article/details/106532357/
https://k8s-install.opsnull.com/08-4.kube-prometheus%E6%8F%92%E4%BB%B6.html
https://github.com/prometheus-operator/kube-prometheus


##### 转载请注明出处，本文采用 CC4.0 协议授权
