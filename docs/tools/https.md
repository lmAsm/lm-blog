---
sticky: 999
description: Certbot—30秒部署你的HTTPS,永久免费,自动续约。
descriptionHTML: ''
tag:
- https
- certbot
top: 1
sidebar: true
date: 2025-01-29
---

# 使用Certbot部署HTTPS
Certbot—30秒部署你的HTTPS,永久免费,自动续约。

## 简介
Certbot—30秒部署你的HTTPS,永久免费,自动续约。

Certbot 是一个由 Let's Encrypt 开发的免费开源工具，用于自动化部署和管理 SSL/TLS 证书。它具有以下几个显著的好处：

- 免费证书：Certbot 使用 Let's Encrypt 作为其证书颁发机构，Let's Encrypt 提供免费的 SSL/TLS 证书。这意味着您可以使用 Certbot 轻松获取和更新有效的证书，而无需支付费用。
- 自动化：可以通过设置定时任务来达到永久自动更新。

## 安装

通过下面命令安装certbot：

```shell
apt-get install certbot
```

然后签发证书：

```shell
certbot certonly --standalone -d lmasm.top
```

> **注意**：执行命令时不能启动 nginx 和 memos 的服务，确保他们为关闭状态。
> 可以通过 `service nginx stop` 来关闭 nginx 服务，使用 `service nginx start`。

证书签发完毕之后会存放在 `/etc/letsencrypt/live/` 目录：

![image-20250126154538301](/Users/admin/Library/Application Support/typora-user-images/image-20250126154538301.png)

此时证书就申请好了。

接下来继续编辑对应的nginx配置，将之前配置的80端口替换为443接口。

```nginx
# memos.example.com.conf  
server {
    listen 443 ssl;
    server_name memos.lmasm.top;
    client_max_body_size 1024m;

    ssl_certificate      /etc/letsencrypt/live/memos.lmasm.top/fullchain.pem;
    ssl_certificate_key  /etc/letsencrypt/live/memos.lmasm.top/privkey.pem;

    ssl_session_timeout  5m;

    ssl_ciphers  HIGH:!aNULL:!MD5;
    ssl_prefer_server_ciphers  on;

    location / {
        proxy_pass http://localhost:5230;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

这样我们的域名就支持 HTTPS 访问了。通过 service nginx start命令来打开nginx后即可访问！

注意：如果访问后出现了welcome to nginx的页面，而不是我们的预期页面，不要慌，这是因为缓存问题。手动将页面url修改为https://lmasm.top后即可访问。

### 定时更新证书

从上图可知，certbot 签发的证书只有三个月，到期后需要重新申请证书，为了自动化，可以设定 crontab 定时任务执行：

```
crontab -e
```

把下面的指令贴到后面即可：

```
0 3 1 * * certbot renew --quiet --pre-hook "service nginx stop" --post-hook "service nginx start"
```

意思是每个月 1 号的凌晨三点执行签发任务，并且执行前关闭 nginx 服务，执行完毕后再开启 nginx。

### 问题处理

![image-20250126155413613](/Users/admin/Library/Application Support/typora-user-images/image-20250126155413613.png)

如果在申请证书提示上图这个错误时，首先排查下80端口是否被占用了。通过`lsof -i :80` 命令检查。如果都关闭了，还有这个问题，那么修改对应的nginx.conf文件，在http的server中添加如下配置:

```shell
location ~ /.well-known {
    allow all;
}
```

再次重新申请证书就没有问题了。

