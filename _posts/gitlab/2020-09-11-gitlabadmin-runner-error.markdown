---
layout: post
title: "gitlab 通过备份还原 admin/runner 不能访问"
subtitle: "gitlab 通过备份还原 admin/runner 不能访问"
author: "duanzh"
header-img: "img/home-bg.jpg"
header-mask: 0.3
tags:
  - gitlab
---

> 本文介绍 gitlab 通过备份还原 admin/runner 500 Internal Server Error，记录如何恢复
 
## # Gitlab还原备份
```
 gitlab-backup restore BACKUP=11493107454_xxxx-ce
```
按照提示一步一步恢复
## # 访问 /admin/runners
问题描述：
通过上面恢复备份后发现，访问runner管理页面出现以下错误。
`状态码500 
提示 （Whoops, something went wrong on our end.）`

## # 问题解决
通过查看官方issue,尝试把`旧的/etc/gitlab/gitlab-secrets.json` -- `copy to 新的gitlab服务器来解决此问题`.
拷贝完成，重启即可恢复。


##### 转载请注明出处，本文采用 CC4.0 协议授权
