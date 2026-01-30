---
date: '2026-01-31T01:40:00+08:00'
draft: false
title: "使用 systemd 优雅的管理自启动服务"
description: '介绍如何使用 systemd 来管理和配置 Linux 系统中的自启动服务。'
author: Cassius0924
cover:
  image: "https://s2.loli.net/2026/01/31/Zrlvk4jgiQbsVy3.jpg"
  alt: "使用 systemd 优雅的管理自启动服务"
  caption: "使用 systemd 优雅的管理自启动服务"
tags: ["Linux", "systemd", "服务管理", "自启动"]
---

# 使用 systemd 优雅的管理自启动服务

systemd 是现代 Linux 发行版中广泛使用的初始化系统和服务管理器。它不仅提供了强大的功能来管理系统服务，还允许用户轻松地配置和管理自启动服务。

systemd 中的字母 d 表示 **（daemon）守护进程**，相信学过操作系统的同学都知道守护进程是指在后台运行的进程。

**systemd 作为守护进程管理器，负责启动、停止和管理系统中的各种服务。**

## systemd 操作命令

### systemctl

systemctl（system control）是 systemd 的主要命令行工具。它用于检查和控制 systemd 系统和服务管理器的状态。

- 刷新配置文件

    ```bash
    sudo systemctl daemon-reload
    ```

    但你修改了服务的配置文件后，需要运行这个命令来让 systemd 重新加载配置文件。否则 systemd 不会识别你的更改。

- 启动服务

    ```bash
    sudo systemctl start <service_name>
    ```

    这个命令用于启动指定的服务，当系统重启后，服务不会自动启动。

- 停止服务

    ```bash
    sudo systemctl stop <service_name>
    ```

    这个命令用于停止指定的服务。

- 重启服务

    ```bash
    sudo systemctl restart <service_name>
    ```

    这个命令用于重启指定的服务。

- 重新加载服务配置

    ```bash
    sudo systemctl reload <service_name>
    ```

    这个命令用于重新加载指定服务的配置，而不停止服务。它与 `restart` 的区别在于，`reload` 不会中断服务的运行，适用于支持热加载配置的服务。

- 查看服务状态

    ```bash
    sudo systemctl status <service_name>
    ```

    这个命令用于查看指定服务的当前状态。

- 启用服务自启动

    ```bash
    sudo systemctl enable <service_name>
    ```

    这个命令用于使指定的服务在系统启动时自动启动。但是不会立即启动服务，如果想立即启动服务，可以加上 `--now` 选项： `sudo systemctl enable --now <service_name>`

- 禁用服务自启动

    ```bash
    sudo systemctl disable <service_name>
    ```

    这个命令用于禁止指定的服务在系统启动时自动启动。

- 查询服务是否启用自启动

    ```bash
    sudo systemctl is-enabled <service_name>
    ```

    这个命令用于检查指定的服务是否设置为自启动。

- 注销服务

    ```bash
    sudo systemctl mask <service_name>
    ```

    这个命令用于禁止指定的服务被启动，包括手动启动和自动启动。应用场景如：某些服务存在安全隐患时，可以使用该命令彻底禁止其运行。

- 取消注销服务

    ```bash
    sudo systemctl unmask <service_name>
    ```

    这个命令用于取消对指定服务的禁止启动设置。

- 查看所有正在运行的服务

    ```bash
    sudo systemctl list-units --type=service --state=running
    ```

    这个命令用于列出所有当前正在运行的服务。

### journalctl

journalctl 是 systemd 的日志查看工具。它用于查看和管理由 systemd 记录的日志信息。

- 查看指定服务的日志

    ```bash
    sudo journalctl -u <service_name>
    ```

    加上 `-f` 参数可以实时查看日志输出。

    如果你也不想输入 `journalctl` 这么长的命令，可以用下面 bash 函数简化：

    ```bash
    srvlog() {
        sudo journalctl -u "$@"
    }
    ```

    我使用 `srvlog <service_name>` 来查看服务日志，这里使用了 `$@`，表示传递给函数的所有参数，所以 `-f` 等参数也可以传递进去。当然你也可以把 `srvlog` 改成你喜欢的名字。

### systemd-analyze

systemd-analyze 是 systemd 的性能分析工具。它用于分析系统启动时间和服务启动时间。

- 查看系统启动时间

    ```bash
    systemd-analyze
    ```

    这个命令显示系统启动所花费的总时间以及内核、initrd 和用户空间的时间。

- 查看各个服务的启动时间

    ```bash
    systemd-analyze blame
    ```

    这个命令列出所有服务及其启动时间，按时间长短排序，帮助识别启动缓慢的服务。

- 查看启动过程的可视化图表

    ```bash
    systemd-analyze plot > boot.svg
    ```

    这个命令生成一个 SVG 文件，显示系统启动过程的可视化图表，便于分析启动过程中的瓶颈。

## 服务配置文件

systemd 的服务配置文件通常位于 `/etc/systemd/system/` 或 `/lib/systemd/system/` 目录下，文件扩展名为 `.service`。这些文件定义了服务的启动方式、依赖关系等信息。

一个简单的服务配置文件示例如下：

```ini
[Unit]
Description=My Custom Service
After=network.target
[Service]
ExecStart=/usr/bin/my_custom_service
Restart=always
[Install]
WantedBy=multi-user.target
```

### 服务配置文件参数

配置文件通常分为三个区块：

1. **`[Unit]`**：描述服务的元数据和依赖关系（启动顺序）。
2. **`[Service]`**：核心区块，描述如何启动、停止、重启以及运行时的环境。
3. **`[Install]`**：描述如何“安装”这个服务（即 `systemctl enable` 时挂载到哪个目标）。

以下是常用的参数详解：

### `[Unit]` 区块（依赖与顺序）

这一块决定了服务“什么时候”启动。

| 参数名 | 意思 | 说明 |
| --- | --- | --- |
| **Description** | 描述 | `systemctl status` 时显示的文本，可写可不写，给自己看的。 |
| **Documentation** | 文档链接 | 可选，http 链接或 man 页面。 |
| **After** | **在谁之后启动** | 仅控制启动顺序，不代表强依赖。例如 `After=network.target` 表示网络栈初始化后再启动我。 |
| **Before** | **在谁之前启动** | 仅控制启动顺序，不代表强依赖。 |
| **Requires** | **强依赖** | 如果这里列出的服务启动失败，那么我也不会启动；如果它中途停了，我也会被停止。 |
| **Wants** | **弱依赖** | 我“想要”它启动，但如果它启动失败了，并不影响我继续运行。最常用的依赖方式。 |


### `[Service]` 区块

这是你最关心的部分，决定了服务怎么跑、挂了怎么救。

| 参数名 | 意思与常见值 | 说明 |
| --- | --- | --- |
| **Type** | **启动类型** | `simple`: （默认）启动的主进程就是服务本身（适合绝大多数程序）。 <br>`forking`: 程序启动后会派生子进程并在后台运行（如 Nginx）。<br> `oneshot`: 执行一次就结束（适合运行脚本）。 |
| **ExecStart** | **启动命令** | **必须使用绝对路径**（例如 `/usr/bin/python3` 而不是 `python3`）。 |
| **ExecStop** | **停止命令** | Systemd 默认会发 `SIGTERM` 信号杀进程。如果你需要执行特定的清理命令或脚本，可以在这里定义。 |
| **ExecReload** | **重载命令** | 执行 `systemctl reload` 时运行的命令。 |
| **ExecStartPre** | **启动前执行** | 在 `ExecStart` 之前运行。常用于清理环境、检查配置或数据库迁移。 |
| **Restart** | **重启策略** | `no`: （默认）退出后不重启。<br> `always`: 无论怎么退出，总是重启（适合守护进程）。<br>`on-failure`: 只有非正常退出（退出码非0）才重启。 <br> 生产环境常用 `on-failure` 或 `always`。 |
| **RestartSec** | **重启间隔** | 挂掉后等待多久再尝试重启。例如 `5s`。防止进程疯狂闪退导致 CPU 飙升。 |
| **User / Group** | **运行用户/组** | 默认是 `root`。 |
| **WorkingDirectory** | **工作目录** | 进程启动时的当前目录。对于 `docker-compose` 很重要，因为它要找当前目录下的 `.yml` 文件。 |
| **Environment** | **环境变量** | 设置环境变量，如 `Environment="ENV=production"`。 |
| **EnvironmentFile** | **环境变量文件** | 从文件中读取环境变量（类似 `.env`），如 `EnvironmentFile=/etc/default/myapp`。 |
| **TimeoutStartSec** | **启动超时时间** | 如果服务启动太慢，超过这个时间 Systemd 会杀掉它。默认 90s。如果是初始化很慢的服务），可以设为 `0` (无限) 或更大值。 |

> [!NOTE]
>
> **Service.Type 的区别**
>
> 主要在于 systemd 如何判断服务已经启动成功：
>
> - `simple`：systemd 认为服务一启动就成功。
> - `forking`：systemd 认为当主进程派生出子进程后，服务就成功启动了。
> - `oneshot`：systemd 认为当程序执行完毕并返回时，服务就成功启动了。


### `[Install]` 区块

这一块决定了 `systemctl enable` 的行为。

| 参数名 | 意思 | 说明 |
| --- | --- | --- |
| **WantedBy** | **挂载目标** | 通常填 `multi-user.target`。意思是“当系统进入多用户模式（正常启动后的状态）时，请启动我”。 |
| **Alias** | 别名 | 给服务起个别名，比如 `systemctl start myapp` 也可以用 `systemctl start shortname`。 |

### service 示例

下面是一个完整的 systemd 服务配置文件示例，假设我们需要开机自启动一个 Docker Compose 管理的服务：

直接创建文件 `vim /etc/systemd/system/example.service`，内容如下：

```ini
[Unit]
Description= My Example Service
# 强依赖 docker 服务
Requires=docker.service
# 弱依赖 nginx 服务
Wants=nginx.service

[Service]
Type=simple
WorkingDirectory=/root/dev-service/example-service

# 启动前先停止
ExecStartPre=/usr/bin/docker-compose down
# 启动命令
ExecStart=/usr/bin/docker-compose up
# 停止命令
ExecStop=/usr/bin/docker-compose down

# 无论如何都重启
Restart=always
# 重启间隔 5 秒
RestartSec=5s

[Install]
# 挂载到多用户目标
WantedBy=multi-user.target
```

配置好服务文件后，执行以下命令使其生效：

```bash
sudo systemctl daemon-reload
sudo systemctl enable --now example
```

这样，每次系统启动时，`example` 服务都会自动启动。查看日志的话可以使用 `sudo journalctl -u example -f`。

