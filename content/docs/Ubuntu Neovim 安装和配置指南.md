# Ubuntu Neovim 安装和配置指南



## 安装

不建议使用`apt`安装，因为`apt`安装的版本总是不是最新版。一些neovim插件依赖于高版本的neovim，因此使用apt安装可能会导致一些插件无法正常使用。

下面介绍安装最新版neovim的方法。

下载安装压缩包：

```bash
wget https://github.com/neovim/neovim/releases/latest/download/nvim-linux64.tar.gz
```

安装：

```bash
tar xzvf nvim-linux64.tar.gz
cp ./nvim-linux64/bin/nvim /usr/bin/
```

测试：

```bash
nvim -v
```

![CleanShot 2023-12-31 at 15.15.57@2x](https://s2.loli.net/2023/12/31/SlKxdfWtwNFCOzy.png)

## nvim配置

### 核心配置

配置 nvim 需要先创建配置文件的文件夹。

```bash
cd ~
mkdir -r .config/nvim
cd .config/nvim
```

nvim 使用 lua 语言作为配置文件语言，新建 init.lua，该文件是 nvim 的配置的入口。

```bash
touch init.lua
```

### 模块化配置

nvim 支持模块化配置，所以可以在 nvim 文件夹下创建多个配置模块：

```bash
mkdir -r lua/core
cd lua/core
```

core 文件夹存放 nvim 的核心配置，例如 nvim 基础配置（options.lua）和快捷键配置（keymaps）：

```bash
touch options.lua keymaps.lua
```

此时，neovim 的配置文件结构如下所示：

```
~
`--.config
   `-- nvim
       |-- init.lua
       |-- lua
       |   |-- core
       |   |   |-- keymaps.lua
       |   |   `-- options.lua
       |   `-- plugins
       |       `-- plugins-setup.lua
       `-- plugin
           `-- packer_compiled.lua
```

回到 init.lua 文件，在 init.lua 中调用刚刚新建的两个模块：

```lua
require("core.options")
require("core.keymaps")
```



## 插件管理

使用 lazy.vim 作为插件管理工具，



## 主题



