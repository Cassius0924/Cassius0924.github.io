---
title: 'Git 免密登录配置指南'
draft: false
date: 2023-03-24T10:00:00+08:00
description: '这是一个简单教程，旨在帮助大家实现免密SSH登录，省去每次输入用户名和密码的烦恼。'
author: 'Cassius0924'
tags: ["Git", "SSH", "Passwordless", "Tutorial"]
---

# Git免密登录配置指南

> 是谁还不会Git SSH免密配置？哦，是你🤪

这是一个简单教程，旨在帮助大家实现免密SSH登录，省去每次输入用户名和密码的烦恼。~~（是真的烦呐！）~~

本教程旨在帮助初学者了解Git免密登录的配置方法，以及介绍HTTPS和SSH协议之间的区别。同时提供具体操作步骤。

## HTTPS和SSH的区别

Git支持两种协议：HTTPS和SSH。两种协议各有优缺点：

- HTTPS协议：使用简单，不需要配置SSH密钥，但相对较慢，且不支持Push操作。
- SSH协议：速度快，支持Push操作，但需要配置SSH密钥。

> 2021年8月13日起，Github不再支持密码身份验证。“Support for password authentication was removed on August 13, 2021.“

因此，如果只是从远程仓库拉取代码，建议使用HTTPS协议；如果需要Push代码到远程仓库，则需要配置SSH密钥，使用SSH协议。

## SSH免密登录配置步骤

> 由于SSH协议更安全和优雅且支持Push操作，因此推荐大家使用SSH免密登录，以下是配置教程。

### 前提

先确保你添加远程 Git 远程仓库时使用的是SSH链接，即`git remote add`时应该使用远程仓库的SSH链接。检查方法：

```bash
git remote -v
```

若显示`git@github.com:...`即为SSH链接，若显示`https://...`则为HTTPS链接，使用一下命令进行修改：

```bash
git remote set-url <repo_name> <ssh_url>
```

> 当然，也可以用`git remote rm`命令先删除远程仓库，再用`git remote add`重新添加。

### 步骤

1. 打开终端，输入以下命令生成SSH密钥：

   ```bash
   ssh-keygen -t rsa -C "your_email@example.com"
   ```
   
   > 别直接粘贴上去啊喂！改改后面的邮箱。

2. 按照提示输入密钥保存路径和密码，建议直接回车使用默认值：

   ```bash
   Generating public/private rsa key pair.
   Enter file in which to save the key (~/.ssh/id_rsa): 
   Enter passphrase (empty for no passphrase): 
   ```

   > 也别设置密码啊，不然Push时又要输入你设定的密码，那就不叫免密啦……

   其中`~/.ssh/`即为你的SSH密钥文件夹的路径。

3. 找到SSH密钥文件夹，打开刚刚生成的公钥文件`id_rsa.pub`，复制其全部内容；

   > 注意：要找公钥文件，公钥文件有.pub后缀。`id_rsa.pub`为公钥文件，`id_rsa`为私钥文件。

4. 在Github上的Settings页面中，点击SSH and GPG keys，然后点击New SSH key；

   ![New SSH key](https://s2.loli.net/2023/03/13/tm3CsOKXyxAoMFB.png)

5. 输入一个标题，将复制的公钥内容粘贴到Key文本框中，然后点击Add SSH key；

   ![Add SSH key](https://s2.loli.net/2023/03/13/A78MBsYwzNK9IZd.png)

### 注意

这个配置是全局配置，配置一次就行，不需要每个项目都配置一遍。

务必记住`git clone`和`git remote add`时候都需要使用SSH链接。

![SSH Link](https://s2.loli.net/2023/03/13/JFzNqnj9YtIsK5B.png)

## 写在最后

> 虽然网上已经很多比这更优秀的教程了，但因为我总忘记具体操作流程，所以想写下来强化记忆。

> 是谁还不会Git SSH免密配置？好吧，不是你👀
