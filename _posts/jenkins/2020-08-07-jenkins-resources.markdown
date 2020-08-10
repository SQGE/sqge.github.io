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

> 本文介绍jenkins 共享库 resources的使用场景示例

# 共享库的目录结构如下
```
(root)
 +- src                     # Groovy source files
 |   +- org
 |       +- foo
 |           +- Bar.groovy  # for org.foo.Bar class
 +- vars
 |   +- foo.groovy          # for global 'foo' variable/function
 |   +- foo.txt             # help for 'foo' variable/function
 +- resources               # resource files (external libraries only)
 |   +- org
 |       +- foo
 |           +- bar.json    # static helper data for org.foo.Bar
```
- src
> src目录应类似于标准Java源目录结构。执行管道时，此目录将添加到类路径中。

- vars
> vars目录托管用于定义可从管道脚本访问的全局变量的脚本。

- resources
> resources目录允许libraryResource从外部库中使用步骤来加载相关联的非Groovy文件。当前内部库不支持此功能。



介绍一下目前我这边使用resources的场景
# resources使用场景
目前我这边使用于resources目录，主要放置dockerfile模板文件，用于构建镜像时，去resources获取相应的dockerfile template。

这样的做法可以减少开发同学，编写Dockerfile的问题，全部由ops去定制Dockerfile

- 特点：标准化Dockerfile,减少重复的Dockerfile
- 不足：需要先对各个类型的业务镜像，编写通用的Dockerfile

# 共享库resources目录结构

下面都是各个不同类型的Dockerfile
```
$ /jenkinslibrary/resources$ tree
.
├── war_dockerfile
└── zip_dockerfile
└── python_dockerfile
└── java_dockerfile
```

# 调用共享库方法获取Dokcerfile

根据不同的服务类型生成不同类型的Dockerfile,这样解决了每个服务仓库需要放置Dockerfile的问题


根据不同的env.srv_type，这边指得是服务的类型（service type）进行匹配，生成不同的Dockerfile

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
# 总结
使用resources大大减少了重复的Dockerfile,如果有更好的使用方式，欢迎私信，大家一起交流

##### 转载请注明出处，本文采用 CC4.0 协议授权
