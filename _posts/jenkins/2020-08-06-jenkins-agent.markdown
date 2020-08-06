---
layout: post
title: "jenkins pipeline: agent动态设置代理标签"
subtitle: ""
author: "duanzh"
header-img: "img/home-bg.jpg"
header-mask: 0.3
tags:
  - jenkins
---



#### jenkins 动态设置agent lable

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
