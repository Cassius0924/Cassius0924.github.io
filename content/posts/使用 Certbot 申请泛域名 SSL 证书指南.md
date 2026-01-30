---
date: '2026-01-28T20:23:00+08:00'
draft: false
title: "使用 Certbot 申请泛域名 SSL 证书指南"
description: '介绍如何使用 Certbot 申请和配置泛域名 SSL 证书的步骤。'
author: Cassius0924
tags: ["Certbot", "SSL 证书", "泛域名"]
---

# 使用 Certbot 申请泛域名 SSL 证书指南

使用 Certbot 申请泛域名证书并不复杂，几个月前搞过，但是又忘了，今天重新搞了一遍，记录一下步骤。

## 前提条件

1. 你需要有一个域名，并且可以管理该域名的 DNS 记录。

> 一般我们都是在 *阿里云* 或 *火山引擎* 等云服务商购买的域名，这里以阿里云为例。

## 申请泛域名证书

### 安装 Certbot

先用 `certbot --version` 检查是否已经安装 Certbot，如果没有安装，下面一句话安装一下：

```bash
sudo apt install certbot -y
```

### 申请证书

泛域名证书需要通过 DNS-01 验证域名所有权，使用以下命令申请，注意将 `*.cassdev.com` 和 `cassdev.com` 替换为你的域名，`example@google.com` 替换为你的邮箱。

> [!NOTE]
> **什么是 DNS-01 验证？**
>
> DNS-01 验证是通过在域名的 DNS 记录中添加一个特定的 TXT 记录来证明你对该域名的所有权。Certbot 会生成这个 TXT 记录的值，你需要将其添加到你的 DNS 提供商的管理控制台中。N
>
>
> **为什么需要验证？**
>
> 这是为了确保只有域名的合法所有者才能申请和使用该域名的 SSL/TLS 证书，从而防止恶意用户冒充域名所有者获取证书。

```bash
certbot certonly --manual --preferred-challenges dns -d "*.cassdev.com" -d "cassdev.com" --email example@google.com --agree-tos
```

命令执行后会返回一个**字符串**，先别急着回车，需要先将该字符串添加到你的域名的 DNS 记录中。

![certbot-dns-01](https://s2.loli.net/2026/01/28/iWsj5xQ9LnFyGR8.png)

### 添加 DNS 记录

去到域名控制台，找到 DNS 解析，添加一个 TXT 记录， 将上一步返回的字符串复制到 TXT 记录的值中，主机记录填写 `_acme-challenge`。

> 阿里云的用户请点击跳转：[阿里云云解析 DNS](https://dnsnext.console.aliyun.com/authoritative/domains)

![add-txt-record](https://s2.loli.net/2026/01/28/pRmC1WdTjvH6cZF.png)

### 等待 DNS 记录生效

如命令返回的提示所言，添加完 TXT 记录后，需要等待一段时间让 DNS 记录生效。

我们可以访问 <https://toolbox.googleapps.com/apps/dig/#TXT/_acme-challenge.yourdomain.com> 来检查你的域名的 acme-challenge TXT 记录是否生效。

> 注意将 `yourdomain.com` 替换为你的域名。

记录生效后，回到命令行按回车继续。成功的话可以看到如下提示：

![success-tip](https://s2.loli.net/2026/01/28/DV3U8TbsPH6fq7Q.png)

### 查找证书文件

证书文件默认存放在 `/etc/letsencrypt/live/yourdomain.com/` 目录下，里面包含以下文件：

- `fullchain.pem`：包含服务器证书和中间证书的文件，通常用于配置服务器。
- `privkey.pem`：私钥文件，必须妥善保管，不能泄露给他人。

至此，泛域名 SSL 证书申请完成，可以在服务器上配置使用了。


## 证书续期

泛域名证书的过期时间为 90 天，不过 Certbot 提供了自动续期功能，可以通过以下命令手动续期：

```bash
certbot renew
```

我相信没人想每三个月手动续期一次，所以建议使用 `cron` 定时任务来自动续期证书。

编辑 `crontab` 文件：

```bash
crontab -e
```

添加以下内容，每三个月的第一天凌晨 3 点执行续期命令，续期成功后重载 Nginx 配置：

```bash
0 3 1 */3 * certbot renew --quiet --post-hook "nginx -s reload"
```

简单解释一下这个 cron 表达式：

|分|时|日| 月 | 周 | 命令 |
|---|---|---|---|---|---|
| 0 | 3 | 1 | */3 | * | certbot renew --quiet --post-hook "nginx -s reload" |


## Nginx 使用泛域名证书示例

下面是一个 Nginx 配置示例，假设你的泛域名是 `*.cassdev.com`，你想为子域名 `abcd.cassdev.com` 配置 SSL：

```nginx
server {
    listen 443 ssl http2;
    listen [::]:443 ssl http2;

    server_name abcd.cassdev.com;

    # SSL 证书路径
    ssl_certificate /etc/letsencrypt/live/cassdev.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/cassdev.com/privkey.pem;

    # 前端静态文件根目录 (请修改为您的实际路径)
    root /var/www/abcd.cassdev.com;
    index index.html;

    location / {
        try_files $uri $uri/ /index.html;
    }

    # 您可以添加其他配置，例如日志、gzip压缩等
    access_log /var/log/nginx/abcd.access.log;
    error_log /var/log/nginx/abcd.error.log;
}
```
