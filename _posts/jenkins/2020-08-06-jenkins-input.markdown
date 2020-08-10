---
layout: post
title: "jenkins pipeline：input交互式发布"
subtitle: "jenkins pipeline：input交互输入发布"
author: "duanzh"
header-img: "img/home-bg.jpg"
header-mask: 0.3
tags:
  - jenkins
---

> 本文介绍如何使用input使用,卡点式发布。根据不同的环境,选择发布到不同的环境示例。


# input主要功能
`stage`的`input` 指令允许你使用`input step`提示输入。 在应用了`options`后，进入`stage` 的`agent`或评估`when`条件前，`stage`将暂停。 如果`input`被批准, `stage`将会继续。 作为`input`提交的一部分的任何参数都将在环境中用于其他`stage`。



> 下面讲一下我这边的实践,主要打包后通过`DEPLOY_ENV`选择.部署到不同的各个环境。


### 通过input人工选择,发布服务到不同的环境

```
pipeline {
    agent any
    stages {
        stage("deploy") {
            steps {
                timeout(time: 1, unit: 'MINUTES') {
                script {
                    env.DEPLOY_ENV = input message: '选择部署的环境', ok: 'deploy',
                        parameters: [choice(name: 'DEPLOY_ENV', 
                        choices: ['test', 'uat', 'prd'], description: '选择部署环境')]

                        switch("${env.DEPLOY_ENV}"){
                            case 'prd':
                                println('deploy prd env')
                                break;

                            case 'uat':
                                println('deploy uat env')
                                break;

                            case 'test':
                                println('deploy test env')
                                break;
                            
                            default:
                                println('error env')

                        }
                    }
                }
            }
        }
    }
}
```
