---
layout: post
title: "jenkins pipeline: agent动态设置代理标签"
subtitle: "jenkins pipeline: agent动态设置代理标签"
author: "duanzh"
header-img: "img/home-bg.jpg"
header-mask: 0.3
tags:
  - jenkins
---

> 本文介绍jenkins pipeline当中如何动态使用 agent label,进行部署构建

#### jenkins 动态设置agent label

```
def agentLabel

if (profile== "test") {
    agentLabel = "test"
} else if (profile== "uat") {
    agentLabel = "uat"
} else {
    agentLabel = "prd"
}

pipeline {
    agent { label agentLabel }
    
    stages{
        stage('test dynamically agent'){
            steps{
                script{
                    println agentLabel
                }
            }
        }
    }
}
```
##### 转载请注明出处，本文采用 CC4.0 协议授权
