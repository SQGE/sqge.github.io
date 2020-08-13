---
layout: post
title: "docker方式部署的gitlab跨版本迁移升级"
subtitle: "docker方式部署的gitlab跨版本迁移升级"
author: "duanzh"
header-img: "img/home-bg.jpg"
header-mask: 0.3
tags:
  - gitlab
---
> 本文介绍docker化安装的Gitlab如何跨多版本进行升级

当前版本是`10.8.7`
> 建议进入容器操作，我尝试过直接在外部执行备份恢复，由于项目多，时间太长，会导致失败

# 一定要先备份当前版本
# 备份当前的Gitlab，用于更新失败恢复
```
docker exec -it gitlab bash 
gitlab-rake gitlab:backup:create

# 默认执行完毕备份文件存在容器的 /var/opt/gitlab/backups

# 备份文件名大概长这样
1594726692_2020_07_14_10.8.7_gitlab_backup.tar

# 恢复命令
docker exec -it gitlab bash 
gitlab-rake gitlab:backup:restore BACKUP=1594726692_2020_07_14_10.8.7
```

`恢复时候如果提示`：
```
Unpacking backup ... tar: 1594726692_2020_07_14_10.8.7_gitlab_backup.tar: Cannot open: Permission denied
tar: Error is not recoverable: exiting now
unpacking backup faile
```
执行以下命令：
```
docker exec -i gitlab chown -R git.git  /var/opt/gitlab/backups/1594726692_2020_07_14_10.8.7_gitlab_backup.tar
```

# 接下来进行升级
[官方升级文档](https://docs.gitlab.com/ce/policy/maintenance.html#upgrade-recommendations)



1、先拉取所需镜像到本地
```
docker pull gitlab/gitlab-ce:10.8.7-ce.0
docker pull gitlab/gitlab-ce:11.3.4-ce.0
docker pull gitlab/gitlab-ce:11.11.8-ce.0
docker pull gitlab/gitlab-ce:12.0.12-ce.0 
docker pull gitlab/gitlab-ce:12.10.6-ce.0
```

2、逐一替换image镜像版本(`直至更新到最新`)
```
10.8.7 -> 11.3.4 -> 11.11.8 -> 12.0.12 -> 12.10.6
```
```
gitlab/gitlab-ce:10.8.7-ce.0
gitlab/gitlab-ce:11.3.4-ce.0
gitlab/gitlab-ce:11.11.8-ce.0
gitlab/gitlab-ce:12.0.12-ce.0 
gitlab/gitlab-ce:12.10.6-ce.0
```

#基本操作流程（每更新一个版本操作一次，以此类推）

- 停止gitlab全部服务
```
docker exec -i  gitlab /opt/gitlab/bin/gitlab-ctl stop
```
停止gitlab容器
```
docker stop gitlab
```
- 更换镜像版本
- 启动服务，确认是否正常
```
docker-compose up
```


# 完整的docker-compose.yml
```
version: '2'
services:
    gitlab:
      image: 'gitlab/gitlab-ce:12.10.6-ce.0'
      container_name: gitlab
      restart: unless-stopped
      hostname: 'gitlab'
      environment:
        TZ: 'Asia/Shanghai'
        GITLAB_OMNIBUS_CONFIG: |
          external_url 'http://10.0.0.100'
          gitlab_rails['gitlab_shell_ssh_port'] = 2212
          gitlab_rails['time_zone'] = 'Asia/Shanghai'
          gitlab_rails['smtp_enable'] = true
          gitlab_rails['smtp_address'] = "smtp.qiye.aliyun.com"
          gitlab_rails['smtp_port'] = 465
          gitlab_rails['smtp_user_name'] = "xx@xx.com"
          gitlab_rails['smtp_password'] = "password"
          gitlab_rails['smtp_authentication'] = "login"
          gitlab_rails['smtp_enable_starttls_auto'] = true
          gitlab_rails['smtp_tls'] = true
          gitlab_rails['gitlab_email_from'] = 'xx@xxcom'
      ports:
        - '80:80'
        - '443:443'
        - '2212:22'
      volumes:
        - config:/etc/gitlab
        - data:/var/opt/gitlab
        - logs:/var/log/gitlab
      logging:
        driver: "json-file"
        options:
          max-file: '3'
          max-size: "20m"
volumes:
    config:
    data:
    logs:
```

# gitlab-runner注册
```
docker run  -t -id -v /data/gitlab-runner/config:/etc/gitlab-runner --name gitlab-runner gitlab/gitlab-runner:v12.10.3

docker exec  -i gitlab-runner \
   gitlab-runner register \
  --non-interactive \
  --executor "shell" \
  --url "http://10.0.0.100/" \
  --registration-token "hGLJCvnzskSEU4MfS2md" \
  --description "devops-runner" \
  --tag-list "build,deploy" \
  --run-untagged="true" \
  --locked="false" \
  --access-level="not_protected"
```
##### 转载请注明出处，本文采用 CC4.0 协议授权
