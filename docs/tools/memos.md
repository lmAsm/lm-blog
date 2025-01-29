---
sticky: 999
description: 10分钟完成自己的memos部署
descriptionHTML: '' 
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

## 优点

- 数据隐私安全。所有数据都安全地存储在本地数据库中。
- 以纯文本形式协作，支持Markdown 语法格式。
- 轻松自定义服务器名称、图标、描述、系统样式和执行脚本，使其成为独一无二的服务器。
- 开源 🦦，有兴趣可以二次开发。
- 最重要的一点，免费使用！

## 部署

使用docker部署，几秒钟即可完成。首先确保自己的服务器上已安装好了docker。

安装docker：

```shell
apt-get install docker
apt-get install docker-compose
```

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

然后使用docker安装memos的镜像。

```
docker run -d --name memos -p 5230:5230 -v ~/.memos/:/var/opt/memos neosmemo/memos:stable
```

`~/.memos/` 目录将用作本地计算机上的数据目录，而 `/var/opt/memos` 是 Docker 中卷的目录，不应修改。`-p` 是指定端口。

当执行完上面的 docker 命令之后，memos 服务就已经启动了，可以使用 `localhost:5230` 来访问了。

### 容器开机自启动

首先查看 docker id：

```
root@admin:~# docker ps -a
CONTAINER ID   IMAGE                   COMMAND     CREATED       STATUS       PORTS                                       NAMES
3f46b74e236e   neosmemo/memos:stable   "./memos"   4 hours ago   Up 4 hours   0.0.0.0:5230->5230/tcp, :::5230->5230/tcp   memos
```

然后通过 update 设置自启动：

```
docker update --restart=always 3f46b74e236e
```

关闭自启动：

```
docker update --restart=no
```

## 设置域名

首先确保服务器安装好了nginx。

```
apt-get install nginx
```

修改nginx.conf文件：

```
# memos.example.com.conf  
server {
    server_name memos.lmasm.top;

    location / {
        proxy_pass http://localhost:5230;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

这样就可以通过自己的域名来访问了。

## 配置优化

1. 字体修改为霞鹜文楷
2. 隐藏一些无用的选项

css 代码如下：

```
/* 修改字体 */
body{font-family: "LXGW WenKai Screen", sans-serif ;}
/* 隐藏 通知 选项卡 */
#header-inbox { display: none;}
/* 隐藏 个人资料 选项卡 */
#header-profile { display: none; }
/* 隐藏 资源库 选项卡 */
#header-resources { display: none; }
/* 隐藏 登录 选项卡 */
#header-auth { display: none; }
```

js代码如下：

```
function changeFont() {
  const link = document.createElement("link");
  link.rel = "stylesheet";
  link.type = "text/css";
  link.href = "https://cdn.staticfile.org/lxgw-wenkai-screen-webfont/1.7.0/lxgwwenkaiscreen.css";
  document.head.append(link);
};
changeFont()
```

## 参考链接

https://imzlp.com/posts/30014/