---
date: '2026-08-05T01:30:00+08:00'
draft: false
title: "把小爱音箱接入 Hermes：打造真正属于你的语音助手"
description: '完整记录如何将小爱音箱 Play 增强版接入自部署的 AI Agent（Hermes），实现"音箱麦克风 → 自研 Agent → 音箱扬声器"的语音闭环，含全部踩坑实录。'
author: Cassius0924
tags: ["AI", "智能音箱", "小爱同学", "Hermes", "语音助手", "智能家居", "Home Assistant"]
---

# 把小爱音箱接入 Hermes：打造真正属于你的语音助手

> 你家的小爱同学，能不能拥有一个真正属于你自己的 AI 大脑？
> 这篇文章完整记录了我把 **小爱音箱 Play 增强版** 接入自部署 Agent（Hermes）的全过程：架构设计、组件拆解、短信验证码登录、多轮会话修复，还有一箩筐踩坑实录。

## 一、为什么要折腾

小爱音箱的"小爱同学"很聪明，但它的聪明是小米云端的黑盒：你无法让它调用你的记忆、控制你的服务器、理解你的专属上下文。而我的服务器上跑着 [**Hermes**](https://hermes-agent.nousresearch.com/docs)——一个自部署的开源 AI Agent，带着持久记忆、工具调用、Home Assistant 控制、飞书接入等完整能力。

目标很简单：

```text
音箱麦克风 → Hermes（我的大脑）→ 音箱扬声器
```

让小爱音箱成为 Hermes 的"耳朵"和"嘴"，客厅里喊一声，动用的却是自己的 Agent。

## 二、硬件前提：为什么走"云端链路"

我的音箱是 **小爱音箱 Play 增强版（型号 L05C）**。调研后发现两个硬件事实：

- **没有 3.5mm AUX IN**（背面只有一个 DC 电源口），无法用线缆把外部音频送进去；
- **蓝牙 A2DP 延迟 100-200ms**，作为语音助手外放体验不佳。

所以**不能走"本地音频接入"路线**（麦克风采集 → 本地 STT → Agent → 本地 TTS → 线缆/蓝牙播放），而是走**云端链路**：小爱音箱本身的拾音和发声都是小米云能力，我们只需要在云端"拦截"它识别出来的文字，把回答"塞回"它的嘴巴。

## 三、架构总览

整条链路共 4 个组件，全部跑在自己的服务器上：

<pre class="mermaid">
flowchart LR
    U[你说话：小爱同学，xxx] --> S[小爱音箱 Play 增强版 L05C]
    S -->|音频| MC[小米云 STT 识别]
    MC -->|识别文本| XG[xiaogpt<br/>监听对话流 + 拦截]
    XG -->|POST /v1/chat/completions| OS[openai-shim:8790<br/>OpenAI 兼容翻译层]
    OS -->|POST /v1/voice| VB[hermes-voice-bridge:8788]
    VB -->|hermes chat -q -Q --continue| H[Hermes Agent<br/>voice profile 独立人格]
    H -->|回复文本| VB
    VB --> OS
    OS --> XG
    XG -->|小米云 TTS 原声| MC2[小米云 TTS]
    MC2 -->|音频| S
</pre>

| 组件 | 角色 | 类比 |
|------|------|------|
| **xiaogpt** | 登录小米云，监听音箱对话流，拦截问题；把回答通过小米云 TTS 播报 | 耳朵 + 嘴 |
| **openai-shim** | 把 xiaogpt 以为在调的 OpenAI API 翻译成 voice-bridge 的协议（约 80 行） | 翻译 |
| **hermes-voice-bridge** | 调用 `hermes chat -q -Q --continue <会话ID>` 跑真正的 Agent | 接线员 |
| **Hermes voice profile** | 独立配置的 Agent 实例：语音人设、精简参数、共享记忆 | 大脑 |

## 四、组件逐个拆解

### 4.1 xiaogpt：耳朵与嘴

[xiaogpt](https://github.com/yihong0618/xiaogpt) 是社区项目，核心能力是**通过小米云的私有协议（mina 实时事件流）监听音箱的对话记录**（[MiService](https://github.com/Yonsm/MiService) 等库是对这套协议的封装）——注意，它不碰音频，拿的是云端识别好的**文字**，所以**不需要 root、不需要改固件**。

关键配置（`/opt/xiaogpt/xiao_config.yaml`）：

```yaml
hardware: L05C
mi_did: "634394844"
use_command: true        # L05C 必须！不支持 mina 事件流，改走 miio 命令轮询
mute_xiaoai: true         # 尝试让小爱闭嘴（L05C 上效果有限，见踩坑#3）
bot: chatgptapi
api_base: "http://127.0.0.1:8790/v1"   # 指向本地 shim，而不是 OpenAI！
```

### 4.2 openai-shim：翻译层

xiaogpt 用标准 OpenAI SDK 调用 `api_base`，而 Hermes 的桥是自定义协议。写一个约 80 行的 stdlib HTTP 服务做翻译：

```python
# 核心逻辑（简化）
def ask_hermes(text):
    req = urllib.request.Request(voice_bridge_url,
        data=json.dumps({"text": text}).encode(),
        headers={"Authorization": f"Bearer {KEY}"})
    return json.loads(urllib.request.urlopen(req).read())

# POST /v1/chat/completions
#  取出 messages 最后一条 user 内容 → ask_hermes() → 包装成 OpenAI 响应
#  支持 SSE 流式格式（内容一次性返回，形式上的流式）
```

这样 xiaogpt 完全无感知，以为自己在调用 OpenAI。

### 4.3 hermes-voice-bridge：接线员

服务器上早已部署的桥服务（一个约 140 行的 stdlib HTTP 服务，代码未公开，想复现可在评论区留言），`POST /v1/voice {"text": "..."}` → 返回 `{"ok": true, "text": "Hermes 回复"}`。关键实现：

```python
subprocess.run(
    [HERMES_BIN, "chat", "-q", text, "-Q", "--continue", session_id],
    capture_output=True, text=True, timeout=120)
# stdout 剥离 "session_id:" 行后即回复文本
```

**⚠️ 注意：这里曾经踩了一个大坑（见踩坑#4）——`hermes -z` 会静默忽略 `--continue`，必须用 `hermes chat -q -Q --continue <会话ID>`。**

### 4.4 voice profile：独立人格

Hermes 支持 **profile 机制**——一套完全独立的 Agent 实例（自己的配置、技能、记忆、会话）。为语音场景建了一个 `voice` profile：

```bash
hermes profile create voice --clone   # 克隆配置和技能
# 关键调整（注意用 HERMES_HOME 指定 profile！）
HERMES_HOME=/root/.hermes/profiles/voice hermes config set agent.max_turns 15
HERMES_HOME=/root/.hermes/profiles/voice hermes config set agent.reasoning_effort low
HERMES_HOME=/root/.hermes/profiles/voice hermes config set memory.write_approval false
```

给它写了专属 SOUL.md，核心铁律：**回答 50 字以内、纯口语、禁止 Markdown 符号**（因为要 TTS 朗读）。记忆则通过**软链共享**主 profile 的 MEMORY.md，这样音箱认识你、记得你的环境。

## 五、登录：与小米风控斗智斗勇

小米账号登录是第一个硬骨头。密码登录返回 `securityStatus: 16`（异地/新设备安全验证），需要**短信验证码**。

<pre class="mermaid">
sequenceDiagram
    participant U as 用户
    participant S as 服务器脚本
    participant M as 小米账号API
    participant P as 手机短信

    S->>M: 密码登录
    M-->>S: securityStatus=16<br/>需要安全验证
    S->>M: 触发短信验证
    M->>P: 发送验证码
    P-->>U: 收到短信
    U-->>S: 把验证码发给助手
    S->>M: 提交验证码
    M-->>S: 通过！返回 token
    S->>S: 保存 .mi.token（含 deviceId）
    Note over S,M: 之后复用已验证的 deviceId<br/>重新登录不再需要验证码
</pre>

社区方案是升级 [**MiService 2.4+**](https://github.com/Yonsm/MiService)（作者刚加了 `otp_callback` 支持短信验证码登录），写一个小脚本等待验证码文件即可：

```python
acc = MiAccount(None, user, password, token_store="/root/.mi.token",
                otp_callback=wait_for_code_file)   # 等 /tmp/mi_otp.txt
await acc.login("micoapi")   # xiaogpt 用 micoapi sid
```

**关键认知：验证过的 deviceId 会被小米记住**——之后新进程复用同一 deviceId 登录，不再触发验证码。这也是服务能自动重启的前提。

## 六、踩坑实录（全是真金白银）

### 坑 1：腾讯云 pip 镜像缺包

服务器 pip 全局指向 `mirrors.tencentyun.com` 内网镜像，`xiaogpt`、`setuptools>=64` 全都装不上。解法：**显式指定官方源**：

```bash
pip install -i https://pypi.org/simple xiaogpt[locked]
```

### 坑 2：miservice_fork 登录失败会删 token

`miservice_fork`（xiaogpt 依赖）在登录异常时调用 `save_token(None)`，**直接把 `~/.mi.token` 文件删掉**！我因此被迫走了两次短信验证流程。教训：**测试登录前先备份 token**（`cp /root/.mi.token /root/.mi.token.bak`），并且别用错误方式反复触发登录（会把小米的短信验证码限流，报 `用户行为被限制`）。

### 坑 3：小爱抢答 + 原声 TTS 限制

L05C 的 `mute_xiaoai` 不完全生效——小爱同学会先自己答一句（通常是"这可把我难住了"），然后才是 Hermes 的回答。根因与 4.1 节同一个：**L05C 不支持 mina 实时事件流**，而 mute 机制依赖 mina 的播放控制接口。我也尝试过用 miio 协议直接发暂停 action（`?3-2`），设备直接 `user ack timeout` 不响应。**结论：固件级限制，无解（除非刷机）。**

另外 L05C 的 TTS 只能用小爱原声，edge-tts 等第三方音色用不了。

### 坑 4：`hermes -z` 静默丢弃 `--continue`（最重要的坑）

一开始桥用 `hermes -z "问题" --continue voice-home` 想保持多轮会话，日志看起来一切正常——**实际上每次问答都是全新会话，多轮记忆从来没生效过**。查源码才发现：oneshot 路径（`_run_and_exit_oneshot`）只传递 model/provider/toolsets/usage_file 四个参数，`--continue` 被**静默丢弃**！

```python
# 源码：hermes_cli/main.py
if getattr(args, "oneshot", None):
    _run_and_exit_oneshot(args.oneshot, model=..., provider=...,
                          toolsets=..., usage_file=...)   # 没有 continue！
```

解法：改用 `hermes chat -q <text> -Q --continue <会话ID>`，且 `--continue` 按"标题或 ID"解析——**传会话 ID 才能稳定命中**（名字匹配要求精确标题）。桥从 voice profile 的 state.db 查最新会话 ID 传入。

### 坑 5：DeepSeek 缓存与成本

多轮会话会无限变长？Hermes 自带**上下文压缩**（`compression.enabled: true`：上下文占用达到模型窗口的 50% 时触发压缩，压缩后约占 20%，并保留最近 20 条消息原样）。另外 DeepSeek API 的提示词缓存有 TTL（约 5 分钟），隔一两天不聊，缓存失效后旧前缀需按全价重新计费。最终方案是**日切会话**：

<pre class="mermaid">
flowchart TD
    Q[音箱来问题] --> L{查 voice 库最新会话}
    L -->|最新会话是今天开的| C[继续它<br/>多轮记忆延续]
    L -->|最新会话是昨天/更早的| N[开新会话<br/>跨零点后第一次聊天]
    C --> D[历史保留在库里<br/>永不删除，随时可查]
    N --> D
</pre>

在桥里判断（而非删库）：桥发现最新会话的 `started_at` 早于今天零点 → 返回空会话 ID（None），本次问答自动开新会话。**历史永不删除**，既控制了会话长度，又保留了可追溯的对话记录。

## 七、使用体验

最终效果：

- **普通对话**："小爱同学，今天天气怎么样" → 全部走 Hermes，回答简短口语化（50 字内、无 Markdown）
- **多轮记忆**：当天连续对话有上下文；第二天自动开新会话
- **原生逃生口**："小爱同学，**小爱**，帮我打开空调" → 显式点名小爱，走原生渠道（音乐、闹钟、设备控制交给小爱自己）
- **全量接管**：xiaogpt 的 `need_ask_gpt` 逻辑被我反转（默认全走 Hermes，仅"小爱"前缀放行原生），这是对 site-packages 的 patch，升级 xiaogpt 后需要重打

三个 systemd 服务托底，重启不丢：

```ini
# /etc/systemd/system/hermes-voice-bridge.service
[Service]
Environment=HOME=/root
Environment=HERMES_HOME=/root/.hermes/profiles/voice
ExecStart=/usr/local/bin/hermes-voice-bridge --host 127.0.0.1 --port 8788
Restart=always
```

`openai-shim` 与 `xiaogpt` 同理，三个服务都配置了 `Restart=always`。

## 八、局限与展望

- **抢答问题**：小爱先答一句的毛病是 L05C 固件限制，根治要刷机（不值得）
- **语音识别依赖小米云**：STT 质量由小米决定（实测"心动的信号7"被识别成"兴奋的信号七"😅）
- **响应延迟**：每次问答都会重新拉起一个全新的 Agent 进程（无常驻服务），完整加载需要 5–30 秒；要真流式体验需要 Agent 常驻 + SSE
- **扩展方向**：多个音箱各开一个会话（voice-living-room、voice-bedroom）；接入 Home Assistant 语音管道；给 voice profile 裁剪技能集进一步提速

## 结语

整套链路跑通后，客厅里的小爱音箱就成了我的 Agent 的延伸——它有我的记忆、能查我的服务器、能控制我的家，而这一切都跑在自己的硬件上。社区方案（xiaogpt）加上一个 80 行的 shim 和一个独立 profile，成本极低，收益是"小爱同学"真正变成了"我的同学"。

如果你也有一台吃灰的小爱音箱，不妨试试。有问题欢迎留言讨论。

<script type="module">
import mermaid from 'https://cdn.jsdelivr.net/npm/mermaid@11/dist/mermaid.esm.min.mjs';
mermaid.initialize({ startOnLoad: true, theme: 'default' });
</script>
