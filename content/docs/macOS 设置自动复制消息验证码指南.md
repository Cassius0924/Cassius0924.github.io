# macOS 设置自动复制短信验证码指南

## 前提

- 接受短信验证码的 SMI 卡必须在上 iPhone
- iPhone 和 macOS 须登录同一个 iCloud



## 打开短信转发

打开 iPhone 的设置，找到 Message 短信 App 的设置。

![IMG_4653](https://s2.loli.net/2023/10/30/qHoz7iTwOhpkuW5.png)

往下拉，找到 Text Message Forwarding 短信转发并点击，打开需要被转发的 Mac 电脑。若你有 iPda，也可以顺便转发至 iPad。

![IMG_4654](https://s2.loli.net/2023/10/30/jbnJeSd87K5IRZP.png)

打开 macOS 的 Message 短信应用，按下快捷键 `command`+`,` 打开设置。打开 iMessage 面板，勾选需要接受的短信来源。

![CleanShot 2023-10-30 at 10.56.26@2x](https://s2.loli.net/2023/10/30/2wzkGOfLyrCbiTQ.png)

## 下载 MessAuto

点击跳转 [Github 下载地址](https://github.com/LeeeSe/MessAuto)

下载自己电脑对应的版本，M系列的芯片下载第一个，Intel芯片下载第二个。

![CleanShot 2023-10-29 at 23.24.22@2x](https://s2.loli.net/2023/10/29/UMl8vyE1OsXCQnP.png)

下载完毕后，解压压缩包会直接得到一个名为 MessAuto.app 的应用程序文件，需要将它拖进 Applicatioin 文件夹里。

打开 Finder，按下快捷键`command`+ `T` 创建新的标签页，再按下 `command`+`shift`+`A`打开 Application 应用程序文件夹。

回到第一个标签页，将 MessAuto.app 拖到 Application 文件夹下。

由于 MessAuto 没有 Apple 的开发者证书，所以 M 系列芯片的电脑不能直接打开无需安装的应用程序，所以需要运行命令：

```bash
xattr -cr /Applications/MessAuto.app
```

运行完毕后方可正常打开 MessAuto。

MessAuto 是一个没有 GUI 的菜单栏应用程序。需要授予完全磁盘访问权限以及辅助功能权限。

![MessAuto](https://s2.loli.net/2023/10/30/lcRFEbIvQzZfVSW.png)

MessAuto 会自动检测你的短信验证码并复制到系统的剪贴板中。

Auto paste 即为自动粘贴短信验证码，Auto return 为粘贴验证码后自动回车，Hide icon 为隐藏 MessAuto 的菜单栏图标，Launch at login 为开机自启动，Listen email 为监听邮件验证码。Float window 为悬浮窗功能。

如果开启了 Listen email，则需要邮件 App 常驻后台，否则无法实时获取到最新的邮件。