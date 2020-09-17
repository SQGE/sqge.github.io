---
layout: post
title: "kubeadm 24h过期后，重新生成的token"
subtitle: "kubeadm 24h过期后，重新生成的token
author: "duanzh"
header-img: "img/home-bg.jpg"
header-mask: 0.3
tags:
  - kubernetes
---

>本文介绍kubeadm token过期后重新生成token，默认token的有效期为24小时，当过期之后，该token就不可用了。

## # 解决办法
##### 第一种方法
```
kubeadm token create --print-join-command
```
#####  第二种方法
```
token=$(kubeadm token generate)
kubeadm token create $token --print-join-command --ttl=0
```
##### 第三种方法（手动生成）
```
# 重新生成新的token
kubeadm  token create
kubeadm  token list

# 获取ca证书sha256编码hash值
openssl x509 -pubkey -in /etc/kubernetes/pki/ca.crt | openssl rsa -pubin -outform der 2>/dev/null | openssl dgst -sha256 -hex | sed 's/^.* //'

# 加入集群
kubeadm join masterIP:6443 --token xxxxxxxxxxx --discovery-token-ca-cert-hash sha256:xxxxxx
```
#### 相关文档：
https://www.cnblogs.com/hongdada/p/9854696.html

##### 转载请注明出处，本文采用 CC4.0 协议授权
