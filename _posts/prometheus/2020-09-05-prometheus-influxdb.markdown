---
layout: post
title: "prometheus 远程存储influxdb"
subtitle: "prometheus 远程存储influxdb"
author: "duanzh"
header-img: "img/home-bg.jpg"
header-mask: 0.3
tags:
  - prometheus
---

> 本文介绍，Prometheus使用Influxdb作为远程存储实践。

Prometheus的存储数据库默认只保留15天的数据，但是对于公司而言，需要对历史数据进行统计分析，容量规划等，希望能够将数据永久存储起来，或者说能够让我们自己将数据进行处理，这边介绍使用Influxdb作为远程存储。

InfluxDB对Prometheus远程读写API的支持将以下HTTP端点添加到InfluxDB：
```
/api/v1/prom/read
/api/v1/prom/write
```
此外，还有一个[`/metrics`端点，](https://docs.influxdata.com/influxdb/v1.8/administration/server_monitoring/#influxdb-metrics-http-endpoint)配置为以Prometheus指标格式生成默认的Go指标。

### [创建目标数据库](https://docs.influxdata.com/influxdb/v1.8/supported_protocols/prometheus/#create-a-target-database)

在您的InfluxDB实例中创建一个数据库，以容纳从Prometheus发送的数据。在下面提供的示例中，`prometheus`用作数据库名称，但是欢迎使用您喜欢的任何数据库名称。

```
CREATE DATABASE "prometheus"
```

### [配置Prometheus](https://docs.influxdata.com/influxdb/v1.8/supported_protocols/prometheus/#configuration)

要使Prometheus远程读写API与InfluxDB一起使用，请将URL值添加到[Prometheus配置文件中](https://prometheus.io/docs/prometheus/latest/configuration/configuration/#configuration-file)的以下设置中：

*   [`remote_write`](https://prometheus.io/docs/prometheus/latest/configuration/configuration/#%3Cremote_write%3E)
*   [`remote_read`](https://prometheus.io/docs/prometheus/latest/configuration/configuration/#%3Cremote_read%3E)

这些URL必须可从正在运行的Prometheus服务器上解析，并使用运行InfluxDB的端口（`8086`默认情况下）。还使用`db=`查询参数包括数据库名称。

#### [示例：Prometheus配置文件中的端点](https://docs.influxdata.com/influxdb/v1.8/supported_protocols/prometheus/#example-endpoints-in-prometheus-configuration-file)

```
remote_write:
  - url: "http://localhost:8086/api/v1/prom/write?db=prometheus"

remote_read:
  - url: "http://localhost:8086/api/v1/prom/read?db=prometheus"
```
#### [使用身份验证读取和写入URL](https://docs.influxdata.com/influxdb/v1.8/supported_protocols/prometheus/#read-and-write-urls-with-authentication)

如果[在InfluxDB上启用了身份验证](https://docs.influxdata.com/influxdb/v1.8/administration/authentication_and_authorization/)，请分别使用和参数向具有读和写特权的InfluxDB用户传递`username`和。`password` `u=` `p=`

##### [启用身份验证的端点的示例](https://docs.influxdata.com/influxdb/v1.8/supported_protocols/prometheus/#examples-of-endpoints-with-authentication-enabled)

```
remote_write:
  - url: "http://localhost:8086/api/v1/prom/write?db=prometheus&u=username&p=password"

remote_read:
  - url: "http://localhost:8086/api/v1/prom/read?db=prometheus&u=username&p=password"
```

### [示例：将Prometheus解析为InfluxDB](https://docs.influxdata.com/influxdb/v1.8/supported_protocols/prometheus/#example-parse-prometheus-to-influxdb)

```
# Prometheus metric
example_metric{queue="0:http://example:8086/api/v1/prom/write?db=prometheus",le="0.005"} 308

# Same metric parsed into InfluxDB
measurement
  example_metric
tags
  queue = "0:http://example:8086/api/v1/prom/write?db=prometheus"
  le = "0.005"
  job = "prometheus"
  instance = "localhost:9090"
  __name__ = "example_metric"
fields
  value = 308
```

##### [完整配置： docker-compose.yml](https://github.com/lework/Docker-compose-file/blob/master/prometheus/docker-compose-influxdb.yaml)

```
version: '3'

volumes:
  prometheus_data: {}

services:
  prometheus:
    image: prom/prometheus:v2.16.0
    container_name: prometheus
    hostname: prometheus
    volumes:
      - /etc/localtime:/etc/localtime:ro
      - prometheus_data:/prometheus
      - ./prometheus/etc/prometheus-influxdb.yml:/etc/prometheus/prometheus.yml
    command:
      - '--config.file=/etc/prometheus/prometheus.yml'
      - '--storage.tsdb.path=/prometheus'
      - '--storage.tsdb.retention.time=1d'
      - '--web.console.libraries=/usr/share/prometheus/console_libraries'
      - '--web.console.templates=/usr/share/prometheus/consoles'
      - '--web.enable-admin-api'
      - '--web.enable-lifecycle'
    ports:
      - 9090:9090
    restart: always

  influxdb:
    image: influxdb:1.7-alpine
    container_name: influxdb
    hostname: influxdb
    environment:
      - INFLUXDB_ADMIN_ENABLED=true 
      - INFLUXDB_ADMIN_USER=${INFLUXDB_ADMIN_USER:-admin}
      - INFLUXDB_ADMIN_PASSWORD=${INFLUXDB_ADMIN_PASSWORD:-admin}
      - INFLUXDB_DB=prometheus
      - INFLUXDB_HTTP_LOG_ENABLED=false
      - INFLUXDB_REPORTING_DISABLED=true
      - INFLUXDB_USER=${INFLUXDB_USER:-prometheus}
      - INFLUXDB_USER_PASSWORD=${INFLUXDB_USER_PASSWORD:-prompass}
    volumes:
      - /etc/localtime:/etc/localtime:ro
      - ./influxdb_data:/var/lib/influxdb:rw
    ports:
      - 8086:8086
    restart: always
```

#### [prometheus-influxdb.yml](https://github.com/lework/Docker-compose-file/blob/master/prometheus/prometheus/etc/prometheus-influxdb.yml)
```
# global config
global:
  scrape_interval:     15s # 拉取targets的默认时间间隔,默认是1m.
  scrape_timeout:      10s # 拉去targets的默认超时时间, 默认10s
  evaluation_interval: 15s # 执行rules的时间间隔,默认是1m.

  external_labels:
    monitor: 'dev-monitor'

# Alertmanager configuration
alerting:

# Load rules once and periodically evaluate them according to the global 'evaluation_interval'.
rule_files:
  - "alerts/*.rules"
  # - "second_rules.yml"

# A scrape configuration containing exactly one endpoint to scrape:
# Here it's Prometheus itself.
scrape_configs:
  # The job name is added as a label `job=<job_name>` to any timeseries scraped from this config.
  - job_name: 'prometheus'
    scrape_interval: 5s
    # metrics_path defaults to '/metrics'
    # scheme defaults to 'http'.
    static_configs:
    - targets: ['localhost:9090']
      labels:
        group: 'prometheus'
  - job_name: 'influxdb'
    static_configs:
    - targets: ['influxdb:8086']

remote_write:
  - url: "http://influxdb:8086/api/v1/prom/write?db=prometheus&u=prometheus&p=prompass"

remote_read:
  - url: "http://influxdb:8086/api/v1/prom/read?db=prometheus&u=prometheus&p=prompass"
```

##### 相关资料:
https://docs.influxdata.com/influxdb/v1.8/supported_protocols/prometheus/
https://github.com/lework/Docker-compose-file/blob/master/prometheus/docker-compose-influxdb.yaml
##### 转载请注明出处，本文采用 CC4.0 协议授权
