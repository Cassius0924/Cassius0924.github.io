# macOS Word 设置论文排版指南

## 正文

字体大小默认12，两端对齐，1.5倍行距。字体类型选择 Latin 文字，将字体改成 Times New Roman，再将字体类型改成 Asian 亚洲文字，将字体改成 SimHei（黑体）。

![CleanShot 2023-10-22 at 21.46.45@2x](https://s2.loli.net/2023/10/22/hzB69WHcKSLDYdT.png)

## 带首行缩进的正文

打开样式面板，点击 New Style。

![CleanShot 2023-11-29 at 19.45.34@2x](https://s2.loli.net/2023/11/29/TacpfhOd6IuNqzJ.png)

输入样式名称“Indent”，继承于 Normal 样式。

![CleanShot 2023-11-29 at 19.46.56@2x](https://s2.loli.net/2023/11/29/ynCkXH7jRTfrBce.png)

左下角选择 Paragraph 面板，将 Special 改成 First line，后面填 0.86cm。 

![CleanShot 2023-10-22 at 21.54.02@2x](https://s2.loli.net/2023/10/22/bmk8YnCLD17cBKH.png)

## 各级标题

### 一级标题

打开 Heading 1 一级标题的设置。

![CleanShot 2023-10-22 at 21.31.53@2x](https://s2.loli.net/2023/10/22/IhjbkadGzHcfZoO.png)

字体大小默认16，加粗，黑色，居中对齐，1.5倍行距。字体类型选择 Latin 文字，将字体改成 Times New Roman，再将字体类型改成 Asian 亚洲文字，将字体改成 SimHei（黑体）。

![CleanShot 2023-10-22 at 20.43.56@2x](https://s2.loli.net/2023/10/22/6WeTgSpLbyVMClD.png)

左下角选择 Paragraph 面板，将 Special 改成 none，取消继承于正文样式（Normal）的值。

![CleanShot 2023-11-29 at 19.04.12@2x](https://s2.loli.net/2023/11/29/l7spaX1t8oTMv6E.png)

左下角选择 Numbering 面板，切换到 Outline Numbered，选择图示的排列格式，点击右下角 Customize。

![CleanShot 2023-10-22 at 20.55.03@2x](https://s2.loli.net/2023/10/22/Hw3aDUyEF8MLzqG.png)

选择 Level 1 一级标题，先将 Number style 改成中文简体数字，然后在上方的“一”前后分别输入“第”和“章”，Follow number with 选择 Space，Link level to style 选择 Heading 1。

![CleanShot 2023-10-22 at 21.06.14@2x](https://s2.loli.net/2023/10/22/hsI5dYwnajeV6Ay.png)

### 二级标题

同样打开 Heading 2 二级标题的设置，除了字体大小13和居左对齐外，其他设置均与一级标题一致。

![CleanShot 2023-10-22 at 21.12.27@2x](https://s2.loli.net/2023/10/22/UDZJ8iATtfsmnvc.png)

打开 Outline Numbering list 编辑面板，选择 Level 2，勾选 Legal style numbering，再勾选 Restart numbering after，选择上一级 Level 1。Follow number with 一样选择 Space。

![CleanShot 2023-10-22 at 21.16.01@2x](https://s2.loli.net/2023/10/22/rE95V1Cei8HwG2K.png)



### 三级标题

三级标题与二级标题同理，字体大小为12。

## 有序列表

### 数字括号列表

新建一个 Style，取个名字，然后样式继承于正文（Normal）样式。

![CleanShot 2023-12-03 at 17.06.14@2x](https://s2.loli.net/2023/12/03/I68TrybLqVaGJRw.png)

打开 Numbering 面板，切换到 Numbered 选项卡，选择括号样式（也有可能是仅右括号样式），接着点击 Customize。

![CleanShot 2023-12-03 at 17.07.48@2x](https://s2.loli.net/2023/12/03/T5I7O4bPk9uWlVt.png)

按下图输入。

![CleanShot 2023-12-03 at 17.11.26@2x](https://s2.loli.net/2023/12/03/oIFxAX1renY7iyc.png)

## 无序列表

## 目录

## 页码

## 图片和表格

### 图片

### 表格

新建一个名为 My Table 的样式，继承于 Normal 样式。改为单倍行距即可。

## 图注和表注



## 头注和尾注

### 头注

在 Styles Pane 里找到 Header，将其字体大小修改成 9。

![CleanShot 2023-11-29 at 20.30.00@2x](https://s2.loli.net/2023/11/29/amsEuq24MbtcQf6.png)

## 代码块

## 应用

## 设置快捷键

在每个样式的设置里都有快捷键设置，位于样式设置面板的左下角里的 Shortcut Key。

![CleanShot 2023-11-29 at 20.08.10@2x](https://s2.loli.net/2023/11/29/9j46ifLUvxyZVCs.png)

先录制快捷键，这里的快捷键可以自己设置，录制完毕后，点击 Assign。

![CleanShot 2023-11-29 at 20.09.58@2x](https://s2.loli.net/2023/11/29/5kblwdiaZHuFAzD.png)

快捷键设置推荐：

- 一级标题（Heading 1）：`command` + `1`
- 二级标题（Heading 2）：`command` + `2`
- 三级标题（Heading 3）：`command` + `3`
- 正文（Normal）：`command` + `0`
- 带缩进的正文（Indent）：`command` + `4`

## 保存样式集

最后介绍一下如何将这个样式集保存起来，方便应用在其他 Word 文件中。

按下 `command` + `,` 打开 Word 设置面板，点击第一列的 Ribbon & Toolbar。

![CleanShot 2023-11-29 at 19.11.01@2x](https://s2.loli.net/2023/11/29/vfGJdpYigoywOjK.png)

切换到 Quick Access Toolbar 选项卡，左边栏选择 Commands Not in the Ribbon，接着找到并选中 Style Set，点击右箭头。最后保存即可。

![CleanShot 2023-11-29 at 19.13.23@2x](https://s2.loli.net/2023/11/29/XfnW4gJsm2cdxRH.png)

保存此样式集。

![CleanShot 2023-11-29 at 19.18.14@2x](https://s2.loli.net/2023/11/29/TERpjfP6zMgqoFX.png)
