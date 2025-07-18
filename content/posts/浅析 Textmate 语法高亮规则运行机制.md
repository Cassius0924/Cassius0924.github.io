---
title: '浅析 Textmate 语法高亮规则运行机制'
date: 2025-07-20T15:47:58+08:00
description: '浅析 Textmate 语法高亮规则运行机制，了解其分词和主题化的工作原理'
author: 'Cassius0924'
tags: ["VSCode", "Syntax Highlighting", "TextMate"]
cover:
  image: "https://s2.loli.net/2025/03/16/mQydfgXDYCuKPLo.png"
  alt: "Textmate 语法高亮规则运行机制"
  caption: "Textmate 语法高亮规则运行机制"
draft: false
---

# 浅析 Textmate 语法高亮规则运行机制

## 1. 语法高亮简介

语法高亮是指在IDE或编辑器中，对文本进行**分词**，即将文本拆解为 Token（标记），每个 Token 都有对应的名称（作用域）进行标记。再配合主题样式规则，对不同名称的 Token 的进行**主题化**，以提高代码的可读性。

> 程序员离不开语法高亮，就像作家离不开标点符号一样。（你可以代入一下使用 txt 文本编辑器写代码的场景）

语法高亮由两个部分组成：

- 分词（Tokenization）：将文本拆解为一系列 Token。
- 主题化（Theming）：对 Token 进行样式渲染，如字体颜色、背景色、加粗等。

我们以 JSON 的语法为例，简单介绍一下语法高亮的过程。

首先分词引擎会对 JSON 文本进行分词，下图是将 JSON 文本进行分词后的结果，其中每个矩形所包括的文本都是一个 Token，每个 Token 都有一个作用域名称，例如 `null` 对应的是 `constant.language.json` 作用域。

![alt text](https://s2.loli.net/2025/03/16/mQydfgXDYCuKPLo.png)

然后主题化引擎会根据 Token 的作用域名称，对 Token 进行样式渲染，例如将 `constant.language.json` 作用域映射为蓝色不加粗字体。那么 `null` 就会被渲染为蓝色不加粗字体。

![alt text](https://s2.loli.net/2025/03/16/cho7NWLtem1Evgf.png)

## 2. 分词的实现方式
目前主流的分词实现方式大致有有以下三种：

- 基于**正则表达式**的分词：Textmate
- 基于**词法分析**的分词：Highlight.js
- 基于**语法树**的分词：Tree-sitter
- (如果有其他，欢迎补充)

本文只讨论 Textmate 的语法高亮规则编写。

Textmate 原是 MacOS 下的一款文本编辑器，其语法高亮规则是基于正则表达式的，但由于其规则简单易懂，且支持多种语言，因此被广泛应用于各种编辑器和IDE中，如 VSCode、Sublime Text 等。JetBrains 的 IDE 也集成了 Textmate Bundle 插件，可以直接导入 Textmate 的语法高亮规则。

![alt text](<CleanShot 2025-03-12 at 17.23.49@2x.png>)

## 3. 语法高亮规则的编写

VSCode 官方有一套关于编写 Textmate 语法高亮规则的文档，包含分词和主题化，详见：[Syntax Highlight Guide](https://code.visualstudio.com/api/language-extensions/syntax-highlight-guide)。

本文不会介绍如何编写 Textmate 分词规则，只会浅析其工作原理。

## 4. Textmate 的分词规则运行机制

Textmate 的语法高亮规则是基于正则表达式的，Textmate 的语法高亮引擎会根据我们定义好的语法高亮规则对文本进行分词。分词的过程是从文本的开头开始，逐个字符地匹配规则，直到匹配到一个规则为止，然后将匹配到的字符标记为某种语法类型，然后继续匹配下一个字符，直到匹配到文本的末尾。

以下面的语法高亮规则为例：

```json
{
  "patterns": [
    {
        "match": "\\bhello\\b",
        "name": "keyword.hello.jtgo"
    },
    {
        "match": "\\w+",
        "name": "string.word.jtgo"
    },
    {
        "begin": "{{",
        "beginCaptures": {
            "0": {
                "name": "expression.begin.jtgo"
            }
        },
        "end": "}}",
        "endCaptures": {
            "0": {
                "name": "expression.end.jtgo"
            }
        },
        "name": "expression.jtgo",
        "patterns": [
            {
                "match": "\\b(len|panic|print|println|min|max)\\b(?=\\()",
                "name": "keyword.builtin-function.name.jtgo"
            }
        ]
    }
  ]
}
```

`patterns` 列表中的每一个 item 都是一个规则，在 Textmate 中被称为 `Rule Key`。

- 当 Textmate 引擎匹配到 `hello` 时，会将其标记为 `keyword.hello` 作用域。
- 当 Textmate 引擎匹配到 `\\w+`，也就是任意单词字符时，会将其标记为 `string.word.jtgo` 作用域。
- 当 Textmate 引擎匹配到 `{{` 时，接着会继续匹配直到匹配到 `}}`，并且将 `{{` 和 `}}` 之间的内容使用子规则（嵌套规则）进行匹配。这个规则的作用域映射如下：
  - `{{` -> `expression.begin.jtgo`
  - `}}` -> `expression.end.jtgo`
  - `{{ print("OK") }}` -> `expression.jtgo`
  - `print` -> `keyword.builtin-function.name.jtgo`
- 对于 `{{` 和 `}}` 之间的内容，当 Textmate 引擎匹配到 `\\b(len|panic|print|println|min|max)\\b(?=\\()` 时，也就是 `(` 之前的 `len`、`panic`、`print`、`println`、`min` 或 `max` 时，会将其标记为 `keyword.builtin-function.name.jtgo` 作用域。

例如下面的文本，经过 Textmate 引擎的分词后，会被标记为如下的 Token：

```
# hello world! {{ print("OK") }}
```

![alt text](https://s2.loli.net/2025/03/16/2cLsUR8WnorwI19.png)


### 4.1 JSON 的分词规则

直接进阶到 JSON 的分词规则，详细规则内容以 [VSCode 的内置 JSON 分词规则](https://github.com/microsoft/vscode/blob/main/extensions/json/syntaxes/JSON.tmLanguage.json)为例：

- 整个文件内容默认会被最外层的 `scopeName` 匹配，既所有的内容都会被标记上 `source.json` 作用域。
> 整个 JSON 文件的作用域是 `source.json`

- 引擎会将第一个字符从最外层的 `patterns` 数组开始匹配，从上至下按顺序匹配每一个规则，直到匹配到一个规则为止。
> JSON 的最外层 `patterns` 只有一个规则 `value` 规则，第一个字符会使用 `value` 的规则进行匹配。

- 对于每个规则，如果规则中未包含 `match` 或 `begin` 和 `end`，则会直接递归匹配 `patterns` 中的规则。反之分为两种情况：
  - 只有 `match` 字段，会尝试匹配当前规则的 `match` 字段，如果匹配成功，则将匹配到的字符标记为 `name` 字段和 `captures` 字段中的作用域，并继续匹配下一个字符。若匹配失败，则会跳出规则，回到 `patterns` 中继续匹配下一个规则。
  - 只有 `begin` 和 `end` 字段，会尝试匹配 `begin` 规则，匹配成功时会继续将匹配到字符标记为 `beginCaptures` 字段中的作用域（`end` 字段同理），如果规则中包含 `patterns` 字段，则下一个字符会使用 `patterns` 中的规则进行匹配，直到匹配到 `end` 规则为止。如果规则中未包含 `patterns` 字段，则会直接匹配 `end` 字段。匹配到 `end` 字段后，会将当前规则匹配到所有的字符都标记上 `name` 作用域，`begin` 所匹配字符和 `end` 所匹配字符之间的内容会额外标记上 `contentName` 作用域。最后会跳出当前递归规则，回到上一层规则继续匹配。
> JSON 的 `value` 规则内只有一个 `patterns` 字段，则会直接递归匹配 `patterns` 中的规则。
> 
> JSON 文件的第一个字符是 `{`，引擎会尝试匹配 `constant` 规则，其中只有一个 `match` 规则，但匹配失败，所以会跳出 `constant` 规则，回到 `value` 的 `patterns` 规则中继续匹配下一个规则。
>
> 以此类推，`number`、`string` 和 `array` 规则都会匹配失败。
> 
> 接着会匹配 `object` 规则， `{` 字符会匹配成功其 `begin` 字段规则，接着下一个字符会使用 `object` 规则中的 `patterns` 规则进行匹配。

- 若未匹配到 `patterns` 中的任何规则，则会继续匹配下一个字符。

下面是 Textmate 解析 JSON 内容过程一步步拆解后的示意图：

![alt text](https://s2.loli.net/2025/03/16/If5ad4oylSNgQBO.png)




## 5. 附录

> **正则表达式温习** 
> 
> ![alt text](https://s2.loli.net/2025/03/16/BO7wjhvunEAQbJe.png)
