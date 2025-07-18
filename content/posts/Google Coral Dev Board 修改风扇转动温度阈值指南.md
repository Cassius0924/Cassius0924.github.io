# Google Coral Dev Board 修改散热器温度阈值指南

Google Coral Dev Board 是一款功能强大的开发板，可用于构建机器学习应用程序。该板配备了 Edge TPU，这是一款专门用于机器学习任务的加速器。Edge TPU 非常高效，但它也可能会产生热量。如果温度过高，Edge TPU 可能会降频或关闭。

## 查看目前 TPU 温度

使用命令查看目前开发板核心 TPU 的温度，输出温度为1000倍的摄氏度。

```shell
cat /sys/class/thermal/thermal_zone0/temp
```

如输出`54000`，则代表当前温度为54摄氏度。

## 修改散热器温度阈值

禁用热管理：

```shell
echo "disabled" > /sys/devices/virtual/thermal/thermal_zone0/mode
```

修改散热器温度阈值为50摄氏度

```shell
echo 50000 > /sys/devices/platform/gpio_fan/hwmon/hwmon0/fan1_target
```
