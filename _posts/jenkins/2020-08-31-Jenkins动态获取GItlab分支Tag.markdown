---
layout: post
title: "jenkins Active Choices 动态获取Gitlab 分支和Tag"
subtitle: "jenkins Active Choices 动态获取Gitlab 分支和Tag"
author: "duanzh"
header-img: "img/home-bg.jpg"
header-mask: 0.3
tags:
  - jenkins
---

> 本文介绍如何使用jenkins Active Choices、Active Choice Reactive Parameter，动态获取Gitlab分支和Tag，并进行关联。

# Choices Parameter

```
srcUrl

ssh://git@yourGitUrl/xx.git
```
# Active Choice Reactive Parameter 
```
def gettag = ("git ls-remote -t -h ${srcUrl}").execute()


def list = gettags.text.readLines().collect { it.split()[1].replaceAll('refs/heads/', '').replaceAll('refs/tags/', '').replaceAll("\\^\\{\\}", '')}

list.sort {a,b ->
return b.compareTo(a)
}


```
##### 转载请注明出处，本文采用 CC4.0 协议授权
