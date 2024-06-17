#  搭建自动化部署的 GitHub.io 个人静态博客指南


本文章主要介绍如何使用 GitHub Actions 实现基于 MkDocs 的 GitHub.io 个人静态博客的自动化部署

本指南主要分为以下几个部分：

- [创建 GitHub 仓库](#创建-github-仓库)
- [创建 MkDocs 项目](#创建-mkdocs-项目)
- [配置 GitHub Actions](#配置-github-actions)
- [部署到 GitHub Pages](#部署到-github-pages)

## 创建 GitHub 仓库

由于我们的博客是托管在 GitHub io 上的，所以我们需要一个 GitHub 仓库来存放我们的 MkDocs 博客。

### 创建仓库

首先，我们需要在 GitHub 上创建一个新的仓库，仓库名可以是 `<username>.github.io`，其中 `<username>` 是你的 GitHub 用户名。

![创建 GitHub 仓库](https://s2.loli.net/2024/06/17/X43nikWmO7hVGrb.png)

### 克隆仓库

然后，我们需要将这个仓库克隆到本地：

```bash
git clone <repository-url>
cd <repository-name>
```

## 创建 MkDocs 项目

### 安装 MkDocs

MkDocs 是一个 Python 项目使用 pip 安装：

```bash
pip install mkdocs
```

我们需要创建一个 MkDocs 项目，可以使用 MkDocs 官方提供的模板来创建。直接在仓库根目录执行以下命令即可：

```bash
mkdocs new .
```

然后，我们需要安装 MkDocs Material 主题。当然我们也可以选择其他主题，可以在 MkDocs 的 [WiKi](https://github.com/mkdocs/mkdocs/wiki/MkDocs-Themes) 查看更多主题。

```bash
pip install mkdocs-material
```

### 配置 MkDocs

MkDocs 的配置文件是 `mkdocs.yml`，我们可以在这个文件中配置 MkDocs 的一些参数，比如主题、导航栏等。 具体的配置可以参考 MkDocs 的[官方文档](https://hellowac.github.io/mkdocs-docs-zh/user-guide/configuration)。

下面是我个人的配置文件示例：

<details>
<summary> mkdocs.yml </summary>

```yaml
site_name: Cassius0924's Blog
site_author: Cassius0924
repo_name: 'Cassius0924/Cassius0924.github.io'
copyright: "Copyright &copy; 2024 - 2024 Chihchou Ho"
theme: #这里使用的是 MkDocs Material 主题
  name: 'material'
  palette:
    primary: 'indigo'
    accent: 'indigo'
  feature:
    tabs: true
  language: 'zh'
extra:
  social:
    - icon: 'fontawesome/brands/github'
      link: 'https://github.com/cassius0924'
    - icon: 'fontawesome/brands/bilibili'
      link: 'https://space.bilibili.com/12873865'
markdown_extensions:
  - admonition # 注解块支持
  - pymdownx.arithmatex # 数学公式的TeX语法支持
  - pymdownx.betterem:
      smart_enable: all
  - pymdownx.caret
  - pymdownx.critic
  - pymdownx.details
  - pymdownx.emoji: # 表情支持
      emoji_generator: !!python/name:pymdownx.emoji.to_svg
  - pymdownx.inlinehilite
  - pymdownx.magiclink
  - pymdownx.mark
  - pymdownx.smartsymbols
  - pymdownx.superfences
  - pymdownx.tasklist: # 任务清单支持
      custom_checkbox: true
  - pymdownx.tilde
  - meta # 元数据支持
extra_javascript:
  - 'https://cdnjs.cloudflare.com/ajax/libs/mathjax/2.7.0/MathJax.js?config=TeX-MML-AM_CHTML'
plugins:
  - search
```

</details>

### 编写 Markdown 文件

在 `docs` 目录下编写 Markdown 文件，也就是将我们以前写的 Markdown 格式的博客文件放在这里，这些文件将会被 MkDocs 转换成静态网页。

其中，`index.md` 是默认的首页文件，当访问网站时，会默认显示这个文件。

### 本地预览

在本地预览 MkDocs 生成的静态网页，可以执行以下命令：

```bash
mkdocs serve
```

然后，打开浏览器访问终端打印的地址，如： `http://127.0.0.1:8000/`。可以进一步根据需求修改 MkDocs 配置，自己满意了就可以进行下一步。

## 配置 GitHub Actions

GitHub Actions 是 GitHub 提供的持续集成服务，我们可以使用它来实现自动化部署。

### 创建 Actions 配置文件

在仓库根目录下创建 `.github/workflows` 目录，并在该目录下创建一个名为 `deploy.yml` 的文件，内容如下：

```yaml
name: Deploy
on:
  push:
    branches:
      - main
jobs:
  build:
    name: Deploy docs to GitHub Pages
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v2
          
      - name: Build
        uses: Tiryoh/actions-mkdocs@v0
        with:
          mkdocs_version: 'latest'
          configfile: 'mkdocs.yml'
          
      - name: Deploy
        uses: peaceiris/actions-gh-pages@v4
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}
          publish_dir: ./site
          publish_branch: gh-pages
```

解释一下这个配置文件，这个配置文件定义了一个名为 `Deploy` 的工作流，当 `main` 分支有代码提交时，会触发这个工作流。

这个工作流包含了三个步骤：

1. `Checkout`：检出代码
2. `Build`：构建 MkDocs 项目
3. `Deploy`：部署到 GitHub Pages（推送到 GitHub 的 `gh-pages` 分支）

### 配置 Secrets

可以发现这个 `deploy.yml` 中使用了一个名为 `GITHUB_TOKEN` 的 secret，这个 secret 是用来推送代码到 GitHub 的 `gh-pages` 分支的。

访问 
[https://github.com/settings/tokens/new](https://github.com/settings/tokens/new)，
或者依次点击 GitHub 的个人 Settings -> Developer settings -> Personal access tokens -> Tokens (classic)。
此页面用于生成 GitHub Access Token。

![生成 GitHub Token](https://s2.loli.net/2024/06/17/DF6REOJjt7iH3Vv.png)

按照上图的配置生成一个有 repo 和 workflow 权限的 token，生成后复制这个这个 token，然后在仓库 Settings 的 Secrets 新建一个 Secret，并将这个 token 粘贴到下图箭头指向处。

![配置仓库 Secret](https://s2.loli.net/2024/06/17/Z4OI1lYcCiXRtmT.png)

## 部署到 GitHub Pages

当我们将代码推送到 GitHub 的 `main` 分支时，GitHub Actions 会自动触发工作流，构建 MkDocs 项目，并将生成的静态网页推送到 `gh-pages` 分支。

由于 GitHub 不允许使用 HTTPS 协议推送代码，所以 Push 前需要将本地仓库的远程仓库地址修改为你远程仓库 SSH 地址：

```bash
git remote set-url origin <ssh-repo-url>
```

然后，将代码推送到 GitHub：

```bash
git add .
git commit -m "Add MkDocs project"
git push origin main
```

等待 GitHub Actions 完成构建，此时还未部署成功，还需进行最后一步设置。

### 设置 GitHub Pages

在 GitHub 仓库的 Settings -> Pages 中，将 Branch 设置为 `gh-pages`，Folder 设置为 `/ (root)`，点击 Save 即可。

![GitHub Pages](https://s2.loli.net/2024/06/17/LaRdUmcpiMw4z2f.png)

此时 GitHub 会再运行一个 GitHub Pages 的 action。等待 action 运行完毕完成后，即可访问 `https://<username>.github.io` 查看你的博客。

## 关于自动化

完成上述步骤后，GitHub.io 就可以实现自动化部署了。每次我们将新博客推送到 `main` 分支时，GitHub Actions 会自动构建 MkDocs 项目，并将生成的静态网页推送到 `gh-pages` 分支，网站也会自动更新。
