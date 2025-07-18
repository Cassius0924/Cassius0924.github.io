# Ubuntu 配置开机自动挂载三星 T7 固态硬盘指南

## 连接硬盘

先将硬盘与主机相连接，然后运行`fdisk`命令查看系统的硬盘分区。

```bash
sudo fdisk -l
```

![fdisk -l](https://s2.loli.net/2023/10/19/kvu6H917eayRX3Y.png)

找到自己连接的硬盘，一般为`/dev/sda1`。

## 查看硬盘UUID

运行命令

```bash
sudo blkid
```

![blkid](https://s2.loli.net/2023/10/19/WTeys8u3ElFNxf2.png)

找到自己硬盘的 UUID 和 TYPE，记录下它们。三星 T7 硬盘默认为兼容性较好的 exfat 格式。

## 配置开机自动挂载

接着修改系统`/etc/fstab`文件。

```bash
vim /etc/fstab
```

在文件最后添加一行：

```
UUID=C65A-E9E1 ~/disk exfat defaults,nofail,utf8,dmask=022,fmask=133 0 0
```

内容格式为：

```
UUID=<UUID> <挂载目录> <硬盘格式> <挂载参数> <是否自动备份> <开机是否自检>
```

挂载参数中可以设置：

- `defauls`：默认挂载参数；

- `dmask=`：目录的默认权限；
- `fmask=`：文件的默认权限；
- `uid=`：挂载硬盘的用户id；
- `gid=`：挂载硬盘的组id；
- `utf8`：字符编码；
- `nofail`：错误忽略，如果硬盘不存在依然正常开机。

设置完毕后保存并退出文件。

## 重启 Ubuntu

```bash
sudo reboot
```

重启后将自动挂载硬盘到指定目录。



