---
layout: post
title: "jenkins pipeline: 共享库resources使用"
subtitle: "jenkins pipeline: 共享库resources使用"
author: "duanzh"
header-img: "img/home-bg.jpg"
header-mask: 0.3
tags:
  - jenkins
---

> 本文介绍jenkins 共享库 resources的基本使用方式

resources目录允许libraryResource从外部库中使用步骤来加载相关联的非Groovy文件。目前内部库不支持此功能。
# resources使用
目前我这边使用于，放置dockerfile模板文件，构建镜像时去resources获取相应的dockerfile template。

特点：标准化Dockerfile,减少重复的Dockerfile
不足：需要先对各个类型的业务镜像，编写通用的Dockerfile

```
$ /jenkinslibrary/resources$ tree
.
├── war_dockerfile
└── zip_dockerfile
└── python_dockerfile
└── java_dockerfile
```

# 共享库方法

根据不同的服务类型生成不同类型的Dockerfile,这样解决了每个服务仓库需要放置Dockerfile的问题

```
// dockerfile模板生成
def generateDockerfile(){
    
    switch("${env.srv_type}"){
        case 'zip':
            def data = libraryResource 'zip_dockerfile'
            data = data.replace("SERVICE_NAME", "${env.JOB_BASE_NAME}")
            writeFile(file: 'Dockerfile', text: data)
            println(data)
            break;
            
        case 'war':
            def data = libraryResource 'war_dockerfile'
            writeFile(file: 'Dockerfile', text: data)
            println(data)
            break;

...

...
            
}
```

#### 转载请注明出处，文章转自[段志辉的博客](https://takingx.com/2020/08/07/jenkins-resources/)
