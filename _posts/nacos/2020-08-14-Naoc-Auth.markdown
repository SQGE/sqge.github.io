---
layout:     post
title:      "nacos 接入grafana不展示数据问题解决"
subtitle:   "nacos 接入grafana不展示数据问题解决"
date:       2020-08-14 11:00:00
author:     "ytduanzh"
header-img: "img/home-bg.jpg"
header-mask: 0.3
catalog:    true
tags:
    - nacos
---

> 本文介绍Nacos接入Prometheus监控，使用Grafana Dashaboard数据不展示问题解决,这里说一下我解决的过程。

# 问题描述
Nacos集群按照导入[官方监控手册](https://nacos.io/zh-cn/docs/monitor-guide.html)接入Prometheus,导入[默认官方的Grafana模板](https://github.com/nacos-group/nacos-template/blob/master/nacos-grafana.json),通过Grafana去展示的时候显示如下：

![](/img/Nacos-Grafana/nacos-grafana-error.png)

错误提示`Templating init failed Datasource named prometheus was not found`
# 问题排查
经过排查Prometheus查询确认数据正常收集，指标也是正常:

![](/img/Nacos-Grafana/prometheus-nacos.png)

# 问题解决

出现这种情况请确认默Grafana Datasources Promtheus的数据源名称。
`我这边是大写，默认官方的模板是小写的数据源。`
![](/img/Nacos-Grafana/promethus-granafa.png)

把官方模板修改为你对应的prometheus数据源Name即可，再重新导入一次，显示以下即是正常
![](/img/Nacos-Grafana/nacos-grafana-true.png)
