---
layout: post
title: "Groovy 排序"
subtitle: "Groovy 排序"
author: "duanzh"
header-img: "img/home-bg.jpg"
header-mask: 0.3
tags:
  - jenkins
---

> 本文介绍Groovy排序方法使用

```
def list =[1,55,28,9,10,23,16]

//升序
list.sort {a,b ->  
 return a.compareTo(b)
}

//降序
list.sort {a,b ->  
 return b.compareTo(a)
}

```
##### 转载请注明出处，本文采用 CC4.0 协议授权
