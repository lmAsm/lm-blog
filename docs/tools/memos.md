---
sticky: 999
description: 10分钟完成自己的memos部署
descriptionHTML: '
<span style="color:var(--description-font-color);">10分钟完成自己的memos部署</span>
<pre style="background-color: #292b30; padding: 15px; border-radius: 10px;" class="shiki material-theme-palenight"><code>
    <span class="line"><span style="color:#FFCB6B;">npm</span><span style="color:#A6ACCD;"> </span><span style="color:#C3E88D;">create</span><span style="color:#A6ACCD;"> </span><span style="color:#C3E88D;">@sugarat/theme@latest</span></span>
    <br/>
    <br/>
    <span class="line"><span style="color:#B392F0;">bun create</span><span style="color:#E1E4E8;"> </span><span style="color:#9ECBFF;">@sugarat/theme</span><span style="color:#E1E4E8;"> </span></span>
</code>
</pre>' 
tag:
 - tools
top: 1
sidebar: true
date: 2024-11-28 23:54:00
---

# memos部署
10分钟完成自己的memos部署

## 简介

memos，一个开源、轻量级的笔记系统。我们可以轻松创建有意义的笔记。支持多平台访问、Markdown语法，可以时刻记录我们的想法和感受。

最喜欢的一点就是，memos的界面非常简洁，很喜欢这种风格。[github地址](https://github.com/usememos/memos)

## 部署

使用docker部署，首先确保自己的服务器上已安装好了docker。

常用docker命令：

```shell
# 列出所有镜像
docker images -a
# 删除镜像
docker rmi image_id
# 拉取镜像
docker pull <image>

# 列出所有容器
docker ps -a
# 停止容器
docker stop container_id
# 启动容器
docker start container_id
# 删除容器
docker rm container_id
# 查看容器的日志
docker logs container_id
```

