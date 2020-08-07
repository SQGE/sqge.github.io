---
layout: post
title: "jenkins pipeline：input交互输入发布"
subtitle: "jenkins pipeline：input交互输入发布"
author: "duanzh"
header-img: "img/home-bg.jpg"
header-mask: 0.3
tags:
  - jenkins
---

> 本文介绍如何使用input的使用选择不同的profile,发布到不同的环境

### 通过input人工选择,发布服务到不同的环境

```
pipeline {
    agent any
    stages {
        stage("test") {
            steps {
                timeout(time: 1, unit: 'MINUTES') {
                script {
                    env.DEPLOY_ENV = input message: '选择部署的环境', ok: 'deploy',
                        parameters: [choice(name: 'DEPLOY_ENV', choices: ['prd', 'uat', 'test'], description: '选择部署环境')]

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
