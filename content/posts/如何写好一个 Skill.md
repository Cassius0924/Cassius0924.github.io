---
title: "如何写好一个 Skill"
date: "2026-05-17"
draft: false
tags: ["AI", "Agent", "Skill"]
description: "从触发、执行、上下文管理和评估四个角度，梳理如何写出真正能提升 Agent 能力的 Skill。"
author: "Cassius0924"
---

## 如何写好一个 Skill？

如果只从表面看，Skill 很容易被理解成更长的 Prompt。但实际用下来，Skill 更像是给 Agent 装上的一块可复用能力模块：它告诉模型**什么时候应该使用这套方法、具体该怎么执行、执行到什么程度算完成、如何验证自己做对了**。
因此，写好一个 Skill 的关键，不是把经验全部塞进 `SKILL.md`，而是把一类任务中真正会影响结果的知识，组织成 Agent 能稳定调用的工作流。
暂时无法在飞书文档外展示此内容

## Skill 是什么？

在 Agent Skill 的定义里，一个 Skill 本质上是一个文件夹，**里面至少包含一个** **`SKILL.md`** **文件**。`SKILL.md` 里会写元信息，比如 `name` 和 `description`，也会写具体的执行说明。除此之外，一个 Skill 还可以附带脚本、参考文档、资产文件等资源。
一个典型结构大概是这样：

```text
my-skill/
├── SKILL.md
├── scripts/
├── references/
└── assets/
```

这个结构背后有一个很重要的设计思想：**渐进式披露**。
Agent 启动时不会把所有 Skill 的完整内容都塞进上下文，而是只读取每个 Skill 的 `name` 和 `description`。当用户任务匹配某个 Skill 的描述时，Agent 才会加载完整的 `SKILL.md`。如果执行过程中还需要更详细的资料，再按需读取 `references/`、`assets/` 或执行 `scripts/` 里的内容。
所以，Skill 的第一个门槛不是“内容写得多不多”，而是：**它能不能在正确的时机被触发**。

## `description` 是触发器，不是简介

Skill 写得再好，触发不了等于零。Agent 只在启动时加载每个 Skill 的 `name` 和 `description`，靠这两行决定"要不要把这个 Skill 拉进来"。**描述写不好，Skill 永远不会被调用**。
很多人会把 `description` 写成 Skill 简介：

```yaml
description: 这个 Skill 用于代码 Review。
```

这类描述太弱了。它只说了 Skill 是什么，却没有告诉 Agent **什么时候该用它**。
更好的写法是把它写成触发条件：

```yaml
description: 当用户要求 review 当前改动、检查 PR diff、评估实现风险、发现潜在 bug、指出缺失测试、确认行为是否符合需求或给出可执行修改建议时使用本 Skill。
```

一个好的 `description` 通常有几个特点：

- 使用“当用户……”这类面向 Agent 决策的表达。
- 描述用户意图，而不是描述内部实现。
- 明确哪些场景应该触发，必要时也说明哪些场景不该触发。
- 足够具体，但不要写成一篇小作文。

这里最容易犯的错误有两个。
第一个是过窄。比如只写“用于检查 PR 代码”，结果用户说“这次改动会不会引入回归”“这个分支能不能合并”时，Skill 可能不会触发。
第二个是过宽。比如写“任何需要代码审查的时候就使用”，结果用户只是想让 Agent 补一个函数、改个代码逻辑或案时，也可能误加载代码评审 Skill，浪费上下文。
因此，`description` 本质上不是摘要，而是**触发条件**。

## `SKILL.md` 要写流程，不要堆原则

一个 Skill 被触发后，Agent 会读取完整的 `SKILL.md`。这时最重要的是让 Agent 能直接行动。
不太好的写法是：

```markdown
## 指导原则

- 谨慎处理。
- 遵循项目既有最佳实践。
- 确认结果正确。
- 使用合适的工具。
```

这些话太抽象，Agent 很难据此改变行为。
更好的写法是：

```markdown
## 工作流程

1. 检查输入文件格式，识别表头、分隔符、编码和行数。
2. 如果文件大于 50MB，使用 `scripts/stream_csv.py` 中的流式解析器。
3. 使用 `references/column_aliases.md` 中的映射关系规范化列名。
4. 将转换结果写入新文件，禁止覆盖原始输入。
5. 在报告完成前运行 `scripts/validate_output.py <输出文件>`。
```

这类内容能直接改变 Agent 的行为，因为它给出了顺序、默认工具、边界条件和验证方式。
写 `SKILL.md` 时，可以优先考虑这几类内容。

### 1. 易错点

易错点是 Skill 里最值得写的部分，也就是那些“不写明，模型就容易猜错”的事情。
例如：

```markdown
- 导出文件中的 `account_id` 字段对应 API 响应中的 `uid`。
- 不要使用 `/health` 验证数据库是否就绪，应使用 `/ready`。
- 不要覆盖源文件，始终写入 `outputs/` 目录。
```

易错点不需要长篇解释，关键是具体、可执行、能阻止常见错误。这块内容需要持续迭代。
所以 skill 不是写完就完了，而是一个随着使用不断进化的活文档。

### 2. 写方法论而不是具体答案

Skill 是教 Agent 怎么解决一类问题，不是给某个具体问题的答案。例如一个日志排查 skill：

- 反例：去 `pod-123` 这台机器 grep Panic，把 14:22 那条stack trace贴出来。—— 换个日志就作废。
- 正例：
  - 从告警里提取 实例名 + 时间窗 + 错误关键词；
  - 在时间窗口 ±5 分钟内 grep 错误关键词，按实例聚合；
  - 输出 "受影响实例数 / 首次出现时间 / 代表性 stack trace" 三栏摘要。

后者任何分析查询都能复用。

### 3. 输出模板

如果任务对输出格式有要求，直接给模板，不要只用自然语言描述。
例如：

````markdown
按以下结构返回结果：

```text
摘要：
[一段话概括]

发现：
- [发现 1]
- [发现 2]

下一步：
1. [行动项]
2. [行动项]
```
````

模型对具体样例和模板的遵循通常比抽象描述更稳定。

### 4. 验证闭环

一个好的 Skill 不应该只告诉 Agent 怎么做，还应该告诉它怎么确认自己做对了。
例如：

```markdown
生成输出后：

1. 运行 `scripts/validate.py outputs/result.json`。
2. 如果验证失败，阅读错误信息并修正输出。
3. 重复执行，直到验证通过。
4. 只有在验证通过后，才能报告任务完成。
```

## 从真实任务里提炼 Skill

写 Skill 最好的原材料，不是凭空让模型生成一份“最佳实践”，而是从真实任务里提炼。
一个常见的过程是：

1. 先让 Agent 完成一类真实任务。
2. 在过程中观察它哪里走偏、哪里需要人工纠正。
3. 让 Agent 把成功路径、必要约束、反复出现的纠正点提炼出来。
4. 再使用 skill-creator 写成 Skill，并用新的任务验证它是否真的减少了错误。

比如我们想沉淀一个代码 Review 的 Skill。
不要新建对话直接写“/skill-creator 请帮我生成一个代码评审的 Skill”。这样生成 skill 通常质量不高，输出格式也无法满足自己的诉求。
更好的方式是，先让 Agent 先真实 review 几个 PR 或 diff。
第一次 review 时，Agent 可能只会泛泛地说：

- 代码结构还可以
- 建议补充测试
- 注意异常处理

然后我们在任务过程中不断纠正它：

- “不要先总结，先列具体风险。”
- “每个问题都要带文件路径和行号。”
- “每个问题都需要按照 Critical、Medium、Low 进行优先级排序”
- ...

![](https://bytedance.larkoffice.com/space/api/box/stream/download/asynccode/?code=ZmJlMWI1Nzk1NjI4NzRkMjNiM2I4YTM5ZmE2MGNjODRfTElmbU9lZVI4cTZEemdhenlMa3ZvS0NoVTZoNGtCUk1fVG9rZW46S0VXSWJjRHJHb1Yyanh4SDlvZGNHZUlRbmlkXzE3NzkwNzQ0NDk6MTc3OTA3ODA0OV9WNA)
直到 Agent 的输出结果符合我们预期后，我们就可以通过 /skill-creator 根据当前会话上下文创建一个 skill。

## 控制上下文

Skill 有一个很容易被忽略的成本：**上下文占用**。
一旦 Skill 被触发，`SKILL.md` 会进入 Agent 的上下文窗口。里面每多一段无关内容，都会和用户当前任务的上下文竞争模型的注意力。

- **写 Agent 不知道的东西**：项目特有的约定、领域工作流、非显然的边界情况、特定工具的用法。
- **不要解释常识**：HTTP 怎么工作、Protobuf 协议是 xxx、什么时候用Redis什么时候用MySQL。
- **大型 Skill 用渐进式披露**：`SKILL.md` 应该保持克制，`SKILL.md`只放每次执行都会用到的内容。详细参考材料（API 文档、错误码表）放进 `references/`，并在主文件里写清什么时候去读它，例如当 API 返回非 200 时读 `references/api-errors.md`。

好的写法是：

```markdown
仅当 API 返回非 200 响应时，读取 `references/api-errors.md`。
当用户要求正式报告时，使用 `assets/report-template.md`。
```

不好的写法是：

```markdown
更多细节见 `references/`。
```

后者的问题是触发条件不明确。Agent 不知道什么时候该读，也不知道读哪一个文件，模型大概率会把所有文件都读一遍，即使无关内容也都会全塞进上下文。

## 什么时候需要脚本？

Skill 不一定要带脚本，但当你发现 Agent 每次都在重复做同一类机械工作时，就应该考虑把它固化为脚本。
适合写进 `scripts/` 的场景包括：

- 把Markdown转成PDF。
- 校验输出 JSON 是否符合诉求。
- 通过外部 API 获取相关数据。（如天气预报 skill 获取天气信息）

脚本的价值在于将模型不擅长的机械一致性工作交给确定性程序。
一个经验判断是：如果你希望 Agent 每次都按完全相同的规则判断某件事，那么它大概率应该是脚本，而不是自然语言说明。

## Skill 的评测

Skill 写完后，如果直接拿用例测试一遍跑通了，就以为 Skill 可用了。但 Skill 的目标是稳定改善一类任务，所以至少要评估两件事。

1. **Skill 是否能在该触发的时候触发，不该触发的时候不触发。**

可以准备一组触发测试：

```json
[
  {
    "query": "帮我清洗这个 sales.csv，并输出收入最高的 3 个月",
    "should_trigger": true
  },
  {
    "query": "CSV 是什么格式？",
    "should_trigger": false
  }
]
```

由于模型行为存在随机性，同一条 query 最好跑多次，观察触发率，而不是只看单次结果。

2. **Skill 是否真的改善了输出质量。**

更可靠的评估方式是做对照实验：

- 不使用 Skill 跑一遍，作为基线。
- 使用 Skill 跑一遍。
- 对比输出质量、耗时、token 成本和错误率。

评估时不要只写“输出要更好”。应该把成功条件拆成可检查的断言：

```json
{
  "assertions": [
    "输出内容是 JSON",
    "输出内容至少包含 3 条候选建议",
    "必须有一条最推荐的建议并说明理由"
  ]
}
```

以上的测试流程不需要我们自己编写和操作，通过 A\\ 的 [skill-creator](https://github.com/anthropics/skills/blob/main/skills/skill-creator/SKILL.md) 这个 skill 就能完成。
![](https://bytedance.larkoffice.com/space/api/box/stream/download/asynccode/?code=ZTlmMGFiOWU5N2ZmNDcwOTAwYWMzNjI5ZmQwZmEwZDlfRlkwNnd1SzBrbmsxSXkyVGRETEtaS2FEYk13ck5GbFNfVG9rZW46WFYzZWJxU2Nkb3AyVUN4aHFaSGMwZVNtbm5iXzE3NzkwNzQ0NDk6MTc3OTA3ODA0OV9WNA)
<br>

## 总结

写好一个 Skill，核心不是“把提示词写长”，而是把真实任务中的经验压缩成 Agent 能调用的能力模块。
一个好的 Skill 至少要做到四件事：

1. **触发准确**：`description` 能让 Agent 在正确场景加载它。
2. **流程清晰**：`SKILL.md` 给出可执行步骤，而不是泛泛原则。
3. **上下文克制**：遵循渐进式披露，核心说明短而准，长资料按需加载。
4. **经过评测**：通过触发测试和输出质量测试证明它真的有用。

## 参考资料

- [Agent Skills Overview](https://agentskills.io/)
- [Best practices for skill creators](https://agentskills.io/skill-creation/best-practices)
- [Optimizing skill descriptions](https://agentskills.io/skill-creation/optimizing-descriptions)
- [Evaluating skill output quality](https://agentskills.io/skill-creation/evaluating-skills)
