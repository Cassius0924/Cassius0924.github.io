# YOLO 目标检测算法学习笔记（一）

## 深度学习经典检测方法

- one-stage（单阶段）：YOLO系列

- two-stage（双阶段）：Faster-Rcnn、Mask-Rcnn系列

![深度学习经典检测方法](https://s2.loli.net/2023/08/28/mX5t61TwqAR3nHB.png)

> Faster-Rcnn：物体检测开山之作。

## one-stage 单阶段检测

优势：速度快，适合做实时检测任务。

缺点：效果不佳。

![one-stage 单阶段检测](https://s2.loli.net/2023/08/28/rFkyvMngZcAJPDo.png)

目标检测的两个主要指标：mAP和FPS。

mAP：检测效果的综合指标。（mAP值越大效果越好）

## two-stage 双阶段检测

优势：效果较好。

缺点：速度较慢、不适合用于视频流的实时检测。

Mask-Rcnn是一个非常实用的通用框架（建议了解）。

 ![two-stage 双阶段检测](https://s2.loli.net/2023/08/28/qcefLK8pBjFnsMa.png) 

