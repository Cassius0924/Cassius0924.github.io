---
date: 2026-08-06T00:00:00+08:00
draft: false
title: "给 Hermes Agent 加上 GitHub Release 自动订阅更新：Argus + Webhook 全自动链路"
description: "从手动检查版本更新到 Argus 轮询 + Webhook 触发的全自动更新链路，包括 hermes update 重置本地 Patch 的备份恢复方案"
author: Cassius0924
tags:
  - Hermes
  - GitHub
  - Webhook
  - Argus
  - DevOps
  - 自动化
---

# 给 Hermes Agent 加上 GitHub Release 自动订阅更新：Argus + Webhook 全自动链路

## 起因：手动检查版本更新的日子

我的主服务器（腾讯云，下文叫 tc）上跑着 Hermes Agent（一个开源 AI Agent 框架），它通过飞书和我日常交流。为了让回复以飞书卡片形式展示，我额外安装了一个第三方插件 `hermes-feishu-streaming-card`（HFC），它通过 monkey-patch 的方式把 Hermes 的纯文本回复改写成交互式卡片。

这套组合有一个很烦的维护成本：**每次都要手动检查有没有新版本**。

某天我发现 HFC 出了新版本（4.2.4 → 4.2.8），手动执行了升级：

```bash
/usr/local/lib/hermes-agent/venv/bin/python -m pip install --upgrade \
  "git+https://github.com/baileyh8/hermes-feishu-streaming-card.git" \
  --index-url https://pypi.org/simple
```

升级本身不难，但暴露了两个痛点：

1. **不检查就不知道有更新**——HFC 三天连发了 4 个小版本（4.2.5 ~ 4.2.8），全靠我隔三差五想起来去 GitHub 看一眼
2. **更隐蔽的坑：`hermes update` 会重置本地代码**。8 月 4 日我执行过一次 Hermes 本体更新，结果把之前手工修复的飞书文件发送 bug（一个本地 `git apply` 的补丁）**整个冲掉了**——上游 main 分支至今没修这个 bug。更新完功能悄悄变坏，这才是最可怕的。

于是决定：**把"检查更新"和"执行更新"都自动化**。

## 先搞清楚：GitHub Release 有哪几种订阅方式

在动手之前，先梳理一下监控一个 GitHub 项目有没有发新 Release 的所有可行方案：

| 方案 | 实时性 | 能触发流程？ | 适用场景 |
|---|---|---|---|
| GitHub Watch → Custom → Releases | 实时邮件 | ❌ 只通知 | 最省事，但邮件易被淹没 |
| RSS（`/releases.atom`） | 分钟级 | ❌ 只通知 | 阅读器订阅 |
| `gh release list` + cron | 取决于 cron | ✅ 可扩展 | 自己搭轮询 |
| **GitHub Webhook（release 事件）** | 实时 | ✅ 直接触发 | **只能配在自己拥有的仓库上** |
| 第三方 SaaS（newreleases.io 等） | 分钟级 | ⚠️ 部分支持 | 不想自托管 |
| **Argus 等自托管监控器** | 分钟级 | ✅ 轮询到变化后主动发 Webhook | 别人仓库的最佳解 |

**关键限制**：GitHub 的 Webhook 只能由**仓库所有者**在仓库 Settings 里配置。我想监控的 `NousResearch/hermes-agent` 和 `baileyh8/hermes-feishu-streaming-card` 都不是我的仓库，GitHub 根本不会把它们的 release 事件推送给我。

所以要么 cron 轮询，要么用一个**第三方监控器替我们轮询、然后把结果伪装成 Webhook 推回来**——后者就是我选的 **Argus**。

## 方案设计

两个监控目标，两种更新敏感度：

- **Hermes 本体**：更新会改代码、重置 patch、重启 gateway，敏感 → **Argus 轮询（30 分钟级）→ Webhook → 全自动更新**
- **HFC 卡片插件**：升级只是 pip 换包 + 重启 sidecar，不敏感 → **每天 0 点 cron 检查一次**，无更新就静默

<pre class="mermaid">
flowchart LR
    subgraph GitHub["GitHub 远程仓库"]
        R1["NousResearch/hermes-agent<br/>(别人的仓库)"]
        R2["baileyh8/hermes-feishu-streaming-card<br/>(别人的仓库)"]
    end

    subgraph TC["tc 主服务器"]
        A["Argus 容器<br/>轮询 GitHub API 每 30 分钟"]
        W["adnanh/webhook<br/>:9002 监听"]
        S["update-hermes.sh<br/>自动更新脚本"]
        G["Hermes Gateway<br/>(systemd 自动拉起)"]
        C["cron 每天 0 点<br/>HFC 升级检查脚本"]
    end

    U["飞书通知"]

    R1 -- "发现新 release" --> A
    A -- "GitHub 风格 Webhook<br/>HMAC 签名" --> W
    W -- "触发" --> S
    S -- "hermes update + 重打 patch" --> G
    S -- "通知结果" --> U
    R2 -- "每天 0 点检查" --> C
    C -- "有新版则升级" --> U
</pre>

## 部署 Argus

Argus（[github.com/release-argus/Argus](https://github.com/release-argus/Argus)）是一个轻量级 release 监控器：定期查询目标仓库的最新版本，发现变化后通过 Gotify/Slack/Webhook 等渠道通知，甚至**自动发送 GitHub 风格的 Webhook** 去触发下游流程（官方例子是触发 AWX 的 Playbook）。

### 镜像与配置（两个坑）

```bash
docker pull releaseargus/argus:latest
```

**坑 1：镜像名不是 `release-argus/argus`**，Docker Hub 上是 `releaseargus/argus`（少一个连字符）。直接 pull 会报 `pull access denied`。

**坑 2：config.yml 必须挂载到容器的 `/app/config.yml`**，而不是数据目录 `/data/`。官方 docker-compose 示例：

```yaml
services:
  argus:
    image: releaseargus/argus:latest
    volumes:
      - /path/to/config.yml:/app/config.yml      # 配置文件
      - /path/to/storage/:/app/data/             # argus.db 数据库
    environment:
      ARGUS_UID: 911
      ARGUS_GID: 911
    ports:
      - 127.0.0.1:8080:8080   # 生产环境务必绑 127.0.0.1（UI 无认证）
    restart: always
```

我第一次挂错成 `/data/config.yml`，启动日志显示"Found 1 services to monitor: release-argus/Argus"——它压根没读我的配置，默默在监控它自己 😅

### 监控配置

```yaml
service:
  hermes-agent:
    options:
      active: true
      interval: 30m              # 每 30 分钟查一次
      semantic_versioning: true
    latest_version:
      type: github
      url: NousResearch/hermes-agent
      access_token: ${GITHUB_ACCESS_TOKEN}   # 提高 API 限额；私有仓库必须
      use_prerelease: false
    webhook:
      hermes-update:
        type: github             # GitHub 风格 Webhook（带 X-Hub-Signature-256）
        url: http://127.0.0.1:9002/hooks/update-hermes
        secret: <随机 hex>
        desired_status_code: 0   # 接受任意 2XX
        max_tries: 3
    dashboard:
      auto_approve: true         # 发现新版本自动发 Webhook，不需要 UI 人工审批
      web_url: https://github.com/NousResearch/hermes-agent/releases/tag/{{ version }}
```

**坑 3：gh CLI 并没有 `gh auth token` 这个命令**。我一开始想用它取 token，输出是 `unknown command "token" for "gh auth"`，还把这段错误文本当成了 token 塞进环境变量，导致 Argus 报 `invalid header field value for "Authorization"`。正确姿势是从配置里提取：

```bash
GHTOKEN=$(grep -oP 'oauth_token: \K\S+' /root/.config/gh/hosts.yml | head -1)
docker run -d --name argus --restart unless-stopped \
  -v /root/cassdev/argus/config.yml:/app/config.yml \
  -v /root/cassdev/argus/data:/app/data \
  -p 127.0.0.1:8080:8080 \
  -e GITHUB_ACCESS_TOKEN="$GHTOKEN" \
  -e TZ=Asia/Shanghai \
  releaseargus/argus:latest
```

启动后日志应显示：

```
INFO: Found 1 services to monitor:
  - hermes-agent
INFO: hermes-agent, Latest Release - "2026.8.3"
```

这里有个细节值得注意：**Argus 首次启动查到当前最新版本时不会立即触发 Webhook**（它需要"版本发生变化"才触发）。这很重要——否则一部署就会被自己监控的仓库"背刺"一次，触发一轮无意义的更新。我实测确认了这一点。

## 接收端：adnanh/webhook + HMAC 签名验证

我本机原本就有一个 `adnanh/webhook` 服务（监听 9002，用于我的 sz-metro-api 项目的 GitHub push 自动部署），直接复用，加一个 hook：

```json
{
  "id": "update-hermes",
  "execute-command": "/root/cassdev/webhook/execute-command/update-hermes.sh",
  "command-working-directory": "/usr/local/lib/hermes-agent",
  "response-message": "Hermes update triggered!",
  "trigger-rule": {
    "match": {
      "type": "payload-hmac-sha256",
      "secret": "<和 Argus 相同的 hex>",
      "parameter": { "source": "header", "name": "X-Hub-Signature-256" }
    }
  }
}
```

Argus 发的是 GitHub 风格 Webhook（payload 带 `X-Hub-Signature-256: sha256=<HMAC>`），adnanh 的 `payload-hmac-sha256` 校验规则正好对得上——两端配同一个 secret 即可。

**上线前一定要测签名验证**。我用 Python 构造了三种请求验证：

| 请求 | 结果 | 脚本是否执行 |
|---|---|---|
| 正确签名 | 200 | ✅ |
| 无签名 | 200 | ❌（不执行，adnanh 该版本行为） |
| 错误签名 | 500 | ❌ |

安全属性成立：**只有知道 secret 的调用方才能触发更新**。注意测试时先把 `execute-command` 临时指向一个 echo 脚本，确认链路通了再换回真实脚本，避免误触发真实更新。

## 核心难点：hermes update 会重置本地 Patch

这是整个方案里最需要小心的一环。

Hermes 的 `hermes update` 本质是 `git pull + 重装依赖`，而我的安装目录里有两类**本地修改**，更新后会被冲掉：

1. **HFC 插件的 monkey-patch**——它往 `gateway/run.py`、`cron/scheduler.py`、`gateway/platforms/base.py` 三个文件里注入 `HERMES_FEISHU_CARD_*` 代码块。Hermes 更新会替换这些文件 → 卡片功能静默失效。
2. **飞书文件发送 bug 的本地修复**——`plugins/platforms/feishu/adapter.py` 里一个 90 行的 `git apply` 补丁（上游没合入）。

所以更新脚本必须按"**备份 → 更新 → 恢复**"的流程设计：

<pre class="mermaid">
flowchart TD
    A["Webhook 触发"] --> B["备份本地 patch<br/>git diff adapter.py &gt; 补丁文件<br/>备份 manifest + recovery.lock"]
    B --> C["hermes update --yes"]
    C --> D{"update 成功?"}
    D -- "否" --> E["飞书推送失败信息"]
    D -- "是" --> F["重打 HFC patch<br/>apply_patch / apply_cron_patch / apply_base_patch"]
    F --> G{"adapter.py 仍有非法分支?"}
    G -- "是" --> H["git apply 恢复修复<br/>(apply --check 先行验证)"]
    G -- "否" --> I["上游已修复，跳过"]
    H --> J["SIGTERM gateway<br/>systemd 自动拉起"]
    I --> J
    J --> K["飞书通知结果<br/>版本 + patch 状态"]
</pre>

核心代码（Python 重打 patch 部分）：

```python
from hermes_feishu_card.install.patcher import (
    apply_patch, apply_cron_patch, apply_base_patch,
)

targets = [
    ("/usr/local/lib/hermes-agent/gateway/run.py", apply_patch),
    ("/usr/local/lib/hermes-agent/cron/scheduler.py", apply_cron_patch),
    ("/usr/local/lib/hermes-agent/gateway/platforms/base.py", apply_base_patch),
]
for path, fn in targets:
    with open(path) as f:
        content = f.read()
    patched = fn(content)
    if patched != content:
        with open(path, "w") as f:
            f.write(patched)
        print(f"PATCHED {path}")
```

adapter 修复的恢复用 `git apply --check` 先探测：如果上游已经合入修复（补丁不再适用），就自动跳过，不产生冲突：

```bash
if grep -q "receive_id=thread_id" plugins/platforms/feishu/adapter.py && [ -s "$PATCH_DIR/hermes-adapter-fix.patch" ]; then
    if git apply --check "$PATCH_DIR/hermes-adapter-fix.patch" 2>/dev/null; then
        git apply "$PATCH_DIR/hermes-adapter-fix.patch"
    else
        echo "upstream may have fixed it, skip"
    fi
fi
```

### 重启 gateway 的技巧（两个场景要分开看）

gateway 是 systemd 服务且配了 `Restart=always`。重启它有两种场景，理由不同：

1. **webhook 触发的更新脚本**（由独立的 adnanh/webhook 服务派生，不在 gateway 进程树内）：用 `systemctl restart` 本身不会杀脚本自己，但直接用 SIGTERM 更轻量——不依赖 systemd 会话管理，且配合 `Restart=always` 约 5 秒自动拉起，行为完全一致：
   ```bash
   GPID=$(systemctl --user show hermes-gateway.service -p MainPID --value)
   /bin/kill -s TERM "$GPID"
   ```
2. **Hermes cron 内的升级脚本**：此时脚本跑在 gateway 的调度器里，`systemctl restart` 会把自己所在的 gateway 杀掉（脚本进程也被带走）；而 SIGTERM + `Restart=always` 则能让 gateway 优雅退出并自动拉起。

**cron 场景还要延迟重启**：立即重启会连调度器一起杀掉，升级报告可能来不及投递。用子 shell 延迟 20 秒再杀：

```bash
( sleep 20; /bin/kill -s TERM "$GPID" 2>/dev/null ) &
```

20 秒足够 cron 收集 stdout 并投递飞书，然后再重启加载新代码。

## HFC 卡片：每天 0 点 cron 检查

HFC 的更新不敏感（pip 换包 + 重启 sidecar，sidecar 是独立进程，重启不影响会话），所以用最简单的 cron。完整脚本（`~/.hermes/scripts/hfc-update-checker.sh`）：

```bash
#!/bin/bash
# HFC (hermes-feishu-streaming-card) release 检查 + 自动升级
# 由 Hermes cron 每天 0 点调用。无更新时静默（空输出），有更新时升级并输出报告。
set -uo pipefail

export XDG_RUNTIME_DIR=/run/user/0

VENV_PY=/usr/local/lib/hermes-agent/venv/bin/python
PIP=/usr/local/lib/hermes-agent/venv/bin/pip
SITE_PKG=/usr/local/lib/hermes-agent/venv/lib/python3.11/site-packages
REPO=baileyh8/hermes-feishu-streaming-card
LOG=/var/log/hfc-update.log

# --- 1. 版本对比 ---
LOCAL=$($PIP show hermes-feishu-streaming-card 2>/dev/null | awk '/^Version/{print $2}')
LATEST=$(curl -sL --max-time 20 "https://raw.githubusercontent.com/$REPO/main/hermes_feishu_card/__init__.py" 2>/dev/null | grep -oP '__version__ = "\K[^"]+')

if [ -z "$LATEST" ]; then
    echo "[$(date '+%F %T')] 获取最新版本失败（网络？），跳过" >> "$LOG"
    exit 0   # 网络失败静默
fi
if [ "$LOCAL" = "$LATEST" ]; then
    exit 0   # 无更新静默
fi

echo "[$(date '+%F %T')] 检测到更新: $LOCAL -> $LATEST" >> "$LOG"

# --- 2. pip 升级（cron 环境 PATH 不含 venv，必须用绝对路径）---
export PIP_ROOT_USER_ACTION=ignore
if ! $PIP install --upgrade "git+https://github.com/$REPO.git" --index-url https://pypi.org/simple >> "$LOG" 2>&1; then
    echo "⚠️ HFC 自动升级失败（pip 报错），详情见 $LOG"
    exit 0
fi

# --- 3. 重启 sidecar（独立进程，安全）---
systemctl --user restart hermes-feishu-card-sidecar.service >> "$LOG" 2>&1 || systemctl restart hermes-feishu-card-sidecar.service >> "$LOG" 2>&1

# --- 4. 判断 hook_runtime.py 是否变化，决定是否重启 gateway ---
NEED_GW=0
curl -sL --max-time 20 -o /tmp/hfc_hook_old.py "https://raw.githubusercontent.com/$REPO/v$LOCAL/hermes_feishu_card/hook_runtime.py" 2>/dev/null
if [ -s /tmp/hfc_hook_old.py ] && ! diff -q /tmp/hfc_hook_old.py "$SITE_PKG/hermes_feishu_card/hook_runtime.py" >/dev/null 2>&1; then
    NEED_GW=1
fi

NEW=$($PIP show hermes-feishu-streaming-card 2>/dev/null | awk '/^Version/{print $2}')

REPORT="✅ HFC 已自动升级：$LOCAL → $NEW"
if [ "$NEW" != "$LATEST" ]; then
    REPORT="⚠️ HFC 升级异常：目标 $LATEST，实际 $NEW（详见 $LOG）"
fi
if [ $NEED_GW -eq 1 ]; then
    GPID=$(systemctl --user show hermes-gateway.service -p MainPID --value 2>/dev/null)
    if [ -n "$GPID" ] && [ "$GPID" != "0" ] && [ "$GPID" != "1" ]; then
        # 延迟重启：先让 cron 把本报告投递出去，再杀 gateway（systemd 自动拉起）
        ( sleep 20; /bin/kill -s TERM "$GPID" 2>/dev/null ) &
        REPORT="$REPORT\n（hook_runtime 有变更，gateway 20 秒后自动重启加载）"
    fi
fi
echo -e "$REPORT"
```

几个关键点：
- **cron 环境 PATH 不含 venv**，所以 `pip`/`hermes` 全部用 `/usr/local/lib/hermes-agent/venv/bin/` 绝对路径，裸 `pip` 会报 command not found
- `[ -s /tmp/hfc_hook_old.py ]` 守卫：curl 下载旧版失败时不能拿空文件去 diff，否则会误判 hook_runtime 变化导致 gateway 被无谓重启
- 只在 `hook_runtime.py` 真的变化时才重启 gateway——如果新版只改了 sidecar 代码（比如 `server.py`），重启 sidecar 就够了，完全不需要惊动 gateway

用 Hermes 的 cron 系统注册（`no_agent` 纯脚本模式，非空 stdout 才推送飞书）：

```bash
hermes cron create "0 0 * * *" --name hfc-release-update-check \
  --script hfc-update-checker.sh --no-agent
```

## 安全与运维

1. **签名是唯一防线**：Webhook secret 用 `openssl rand -hex 32` 生成，只存本机文件并限制权限。注意 Argus 容器内以 `911` 用户运行（启动日志 "Applying perms"），配置文件要保证 911 可读——目录设 700、文件 644 即可，别 `chmod 600` 后属主还是 root，那容器就读不到了。
2. **UI 不暴露公网**：Argus Web UI 无认证，只绑定 `127.0.0.1:8080`，远程访问走 SSH 隧道（`ssh -L 8080:127.0.0.1:8080 user@server`）。如果要用，建议放到 WireGuard 内网。
3. **日志落盘**：更新脚本所有输出写 `/var/log/hermes-update.log`，飞书通知只发结果摘要。
4. **失败不静默**：`hermes update` 失败、pip 失败都会通过 `hermes send`（Hermes 自带 CLI 发送通道，不依赖 gateway 运行）推送到飞书。

## 最终效果

```
之前：想起来才去 GitHub 看 → 手动 pip upgrade → 手动重启 → 忘记检查就一直落后
现在：
  Hermes 本体：Argus 每 30 分钟查 release → 发现新版 → 签名 Webhook → 自动更新
              + 重打全部 patch + 重启 gateway → 飞书告知结果
  HFC 卡片：每天 0 点检查 → 有新版自动升级 → 无更新静默
```

整套方案已稳定运行。**最大的收益不是省了那几分钟手动操作，而是把"更新后功能悄悄变坏"这个隐患变成了"更新后自动恢复 + 结果可见"**——patch 有没有重打成功、adapter 修复有没有恢复，每次更新后飞书通知里都写得清清楚楚。目前 HFC 检查链路已实际跑了一段时间，Argus 链路刚上线（触发过一次全链路签名测试），两条链路都会在新版本发布时第一时间给出结果。

如果你也要给一套"别人仓库的版本更新 + 本地有 patch"的组合做自动化，这套 Argus + Webhook + 备份恢复的链路可以直接抄作业。

---

<script type="module">
import mermaid from 'https://cdn.jsdelivr.net/npm/mermaid@11/dist/mermaid.esm.min.mjs';
mermaid.initialize({ startOnLoad: true, theme: 'default' });
</script>
