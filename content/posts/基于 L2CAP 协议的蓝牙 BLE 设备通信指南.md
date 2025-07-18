---
title: '基于 L2CAP 协议的蓝牙 BLE 设备通信指南'
draft: false
date: 2024-06-16T11:20:31+08:00
description: ''
author: 'Cassius0924'
tags: ["Bluetooth", "BLE", "L2CAP", "Socket", "C++", "macOS", "Tutorial"]
---

# 基于 L2CAP 协议的蓝牙 BLE 设备通信指南

## 蓝牙 BLE 是什么

蓝牙BLE，即蓝牙低功耗 （Bluetooth Lower Energy）是一种蓝牙通信标准，设计用于短距离通信和低功耗应用。

相比经典蓝牙，BLE 更加节能，传输距离更远，连接更快。BLE 主要用于健身设备、医疗设备、家居自动化等场景。

## 蓝牙 BLE 设备的连接

## 信道

L2CAP的基本概念是信道（Signaling Channel）。信道是个抽象概念，表示两个设备某个协议层之间的通道。每个信道分配一个2字节的信道ID——CID（Channel ID），每个信道功用不同，比如CID=0x0004的信道表示属性协议（Attribute Protocol）专用信道。对于BLE协议，L2CAP共有三个信道ID：

- 0x0004 – 属性协议
- 0x0005 – 低功耗信令信道
- 0x0006 – 安全管理协议

其他信道则用于经典蓝牙。协议复用可以理解为，不同的协议走不同的信道，互不干扰。

## 代码

```c++
#define ATT_CID 4;

// 创建 L2CAP socket
int s = socket(PF_BLUETOOTH, SOCK_SEQPACKET, BTPROTO_L2CAP);

// 绑定 L2CAP socket
struct sockaddr_l2 bind_addr = {0};
bind_addr.l2_family = AF_BLUETOOTH;
bind_addr.l2_cid = htobs(ATT_CID); // ATT 信道 CID
bdaddr_t any_addr = {{0, 0, 0, 0, 0, 0}};
bacpy(&bind_addr.l2_bdaddr, &any_addr);
bind_addr.l2_bdaddr_type = BDADDR_LE_PUBLIC;
int err = bind(s, (struct sockaddr *)&bind_addr, sizeof(bind_addr));
if (err) {
    return -1;
}

// 连接 L2CAP socket
struct sockaddr_l2 conn_addr = {0};
conn_addr.l2_family = AF_BLUETOOTH;
conn_addr.l2_cid = htobs(ATT_CID); // ATT CID
str2ba(mac_address.c_str(), &conn_addr.l2_bdaddr);
conn_addr.l2_bdaddr_type = BDADDR_LE_PUBLIC;
err = connect(s, (struct sockaddr *)&conn_addr, sizeof(conn_addr));
if (err) {
  exit(-1);
}
```

## L2CAP 数据包解析

L2CAP（Logical Linked Control and Adaptation Protocol 逻辑链路控制与适配协议）工作在链路层,为上层协议提供数据通道。它支持数据分片与重组,确保数据完整可靠地传输。

它位于BLE协议的主机（Host）部分，承担着协议复用（Protocol Multiplex）的任务。

### MTU

MTU，即最大传输单元（Maximum Transmission Unit）。BLE协议默认的MTU为23字节。MTU包括三个部分：**OP Code（1Byte），Handler（2Byte），Payload**。

| OP Code (1 byte) | Handle (2 byte 小端) | Payload (0 - 20 byte) |
| ---------------- | -------------------- | --------------------- |

### OP Code 属性操作码

下表为各个  OP Code 属性操作码的名称和参数。其中以 REQ (Requset) 结尾的为请求信息，以 RSP (Response) 结尾的为回应信息。

| 属性 PDU 名称              | 属性 Opcode | 参数                                       |
| -------------------------- | ----------- | ------------------------------------------ |
| ATT_ERROR_RSP              | 0x01        | 请求操作码错误、属性句柄错误、错误代码     |
| ATT_EXCHANGE_MTU_REQ       | 0x02        | 客户端接收 MTU                             |
| ATT_EXCHANGE_MTU_RSP       | 0x03        | 服务器接收 MTU                             |
| ATT_FIND_INFORMATION_REQ   | 0x04        | 起始句柄、结束句柄                         |
| ATT_FIND_INFORMATION_RSP   | 0x05        | 格式、信息数据                             |
| ATT_FIND_BY_TYPE_VALUE_REQ | 0x06        | 起始 Handle，结束 Handle，属性类型，属性值 |
| ATT_FIND_BY_TYPE_VALUE_RSP | 0x07        | Handle 信息列表                            |
| ATT_READ_BY_TYPE_REQ       | 0x08        | 起始 Handle，结束 Handle，UUID             |
| ATT_READ_BY_TYPE_RSP       | 0x09        | 长度，属性数据列表                         |
| ATT_READ_REQ               | 0x0A        | 属性 Handle                                |
| ATT_READ_RSP               | 0x0B        | 属性值                                     |
| ATT_READ_BLOB_REQ          | 0x0C        | 属性 Handle，值偏移                        |
| ATT_READ_BLOB_RSP          | 0x0D        | 零件属性值                                 |
| ATT_READ_MULTIPLE_REQ      | 0x0E        | Handle 集合                                |
| ATT_READ_MULTIPLE_RSP      | 0x0F        | 值集合                                     |
| ATT_READ_BY_GROUP_TYPE_REQ | 0x10        | 起始 Handle，终止 Handle，UUID             |
| ATT_READ_BY_GROUP_TYPE_RSP | 0x11        | 长度，属性数据列表                         |
| ATT_WRITE_REQ              | 0x12        | 属性 Handle，属性值                        |
| ATT_WRITE_RSP              | 0x13        | ——                                         |
| ATT_WRITE_CMD              | 0x52        | 属性 Handle，属性值                        |

> 详见：[蓝牙核心规范版本 5.3 |第 3 卷，F 部分：3.4.8 Attribute Opcode summary](https://www.bluetooth.com/specifications/specs/core-specification-5-3/)

### Handle 属性句柄

使用 gatttool 命令行工具蓝牙BLE设备的 Handle 信息

```shell
gatttool -i hci0 -b <蓝牙BLE设备MAC地址> --characteristics
```

```shell
gatttool -i hci0 -b <蓝牙BLE设备MAC地址> --primary
```

### Payload 实际数据

Payload 实际数据就是你所需要发送的数据包

## 代码示例

```c++
#define ATT_CID 4;

int ble_connect_l2cap(string mac_address) { // arm mac address: 08:B6:1F:C1:DB:1A
    int s = socket(PF_BLUETOOTH, SOCK_SEQPACKET, BTPROTO_L2CAP);
    if (s < 0) {
        return -1;
    }

    struct sockaddr_l2 bind_addr = {0};
    bind_addr.l2_family = AF_BLUETOOTH;
    bind_addr.l2_cid = htobs(ATT_CID); // ATT CID
    bdaddr_t any_addr = {{0, 0, 0, 0, 0, 0}};
    bacpy(&bind_addr.l2_bdaddr, &any_addr);
    bind_addr.l2_bdaddr_type = BDADDR_LE_PUBLIC;

    int err = bind(s, (struct sockaddr *)&bind_addr, sizeof(bind_addr));
    if (err) {
        Debug::CoutError("{}，绑定L2CAP socket失败", mac_address);
        return -1;
    }

    struct sockaddr_l2 conn_addr = {0};
    conn_addr.l2_family = AF_BLUETOOTH;
    conn_addr.l2_cid = htobs(ATT_CID); // ATT CID
    str2ba(mac_address.c_str(), &conn_addr.l2_bdaddr);
    conn_addr.l2_bdaddr_type = BDADDR_LE_PUBLIC;

    err = connect(s, (struct sockaddr *)&conn_addr, sizeof(conn_addr));
    if (err) {
        Debug::CoutError("{}，连接L2CAP socket失败", mac_address);
        return -1;
    }

    // MTU默认23字节： op code(1 字节)，handle(2 字节，小端)，payload(0-20字节)
    // char on[] = {0x12, 0x2d, 0x00, 0xFE, 0xFE, 0x0F, 0x22, 0x00, 0x00, 0x00, 0x00,
    //              0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x1E, 0xFA};
    char op[] = {0x12};
    char ha[] = {0x2d, 0x00};
    char on[] = {0xFE, 0xFE, 0x0F, 0x22, 0x00, 0x00, 0x00, 0x00, 0x00,
                 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x1E, 0xFA};
    ble_send_l2cap(s, (unsigned char *)op, (unsigned char *)ha, (unsigned char *)on);

    return s;
}

// MTU默认23字节： op code(1 字节)，handle(2 字节，小端)，payload(0-20字节)
int ble_send_l2cap(int fd, const unsigned char *op_code, const unsigned char *handle,
                             const unsigned char *data) {
    // 将 op code, handle, data 拼接成一个数组，使用拼接函数
    unsigned char buf[23];
    memcpy(buf, op_code, 1);
    memcpy(buf + 1, handle, 2);
    memcpy(buf + 3, data, 20);
    int len = -1;
    if (len = write(fd, buf, sizeof(buf)) < 0) {
        Debug::CoutError("发送数据失败");
        return -1;
    }
    return len;
}
```



## 参考文章

- [BLE协议栈 – L2CAP](https://bbs.wireless-tech.cn/d/101-ble-l2cap)
- [Maximum Packet Size According to MTU – KBA203312](https://community.infineon.com/t5/Knowledge-Base-Articles/Maximum-Packet-Size-According-to-MTU-KBA203312/ta-p/252557)
