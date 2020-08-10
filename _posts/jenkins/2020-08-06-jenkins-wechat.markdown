---
layout: post
title: "jenkins pipeline: 企业微信构建通知"
subtitle: "jenkins pipeline: 企业微信构建通知"
author: "duanzh"
header-img: "img/home-bg.jpg"
header-mask: 0.3
tags:
  - jenkins
---

> 本文介绍jenkins pipeline 构建后给企业微信实现自定义通知

# 自定义通知器的实现
>[企业微信API](https://work.weixin.qq.com/api/doc/90000/90003/90487)

步骤:
- 企业微信创建群机器人
- 调用共享库方法

### 共享库wechat 方法
```
// 微信通知

def SendMessage(product,profile,srv_name,status){
    wrap([$class: 'BuildUser']){
        def WechatHook = "https://qyapi.weixin.qq.com/cgi-bin/webhook/send?key=9c69012a-3xxxxxxxxxxx-83b3axxxxx9"

        def reqBody = """{
    	    "msgtype": "markdown",
    	    "markdown": {
                "content": "### 构建信息:\n>- 产品线: **${product}**\n>- 发布环境:  **${profile}**\n>- 应用名称:  **${srv_name}**\n>- 构建发起人:  **${env.BUILD_USER}**\n>-  构建结果: **${status}**\n>- 持续时间: **${currentBuild.durationString}**\n- 构建日志:  [点击查看详情](${env.BUILD_URL})"
                    }
            }"""
    
        
        httpRequest acceptType: 'APPLICATION_JSON_UTF8', 
                consoleLogResponseBody: false, 
                contentType: 'APPLICATION_JSON_UTF8', 
                httpMode: 'POST', 
                ignoreSslErrors: true, 
                requestBody: "${reqBody}", 
                responseHandle: 'NONE', 
                url: "${WechatHook}",
                quiet: true
    }
}

```
### 构建后调用
```
post {
        
    success{
        script{
          if (profile == 'prd') {
            wechat.SendMessage("${product}","${profile}","${srv_name}","构建成功 ✅")
            }
        }
    }
    failure{
        script{
          if (profile == 'prd') {
            wechat.SendMessage("${product}","${profile}","${srv_name}","构建失败 ❌")
            }
        }
    }
    always {
        script {
            
            cleanWs notFailBuild: true 
                
            }
        }
    }
}

```

##### 转载请注明出处，本文采用 CC4.0 协议授权
