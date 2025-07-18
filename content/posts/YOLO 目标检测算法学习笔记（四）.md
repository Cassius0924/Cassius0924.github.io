# YOLO 目标检测算法学习笔记（四）

## YOLO-V2

下图为YOLO-V2相较于YOLO-V1的改进点，以及改进后 mAP 值的变化。

![YOLO-V2](https://s2.loli.net/2023/08/30/CmvK9W1bdpauPtR.png)

## Batch Normalization

V2 版本舍弃了 Dropout，不再有全连接层（Fully connected layers，FC）。每次卷积后都加入 Batch Normalization，对网络的每一层的输入都进行归一化，使收敛更容易。

经过 Batch Normalization 处理后的网络会提升2%的mAP值。

从现在的角度来看，Batch Normatlization 已经称为卷积神经网络处理必备处理了。

## High Resolution Classifier

High Resolution Classifier，即高分辨率分类器，高分辨率分类器。

在 V1 版本，训练时用的是224\*224分辨率的图片，测试时又使用448\*448分辨率的图片，这会导致模型“水土不服”。

针对这一问题，V2 版本在模型训练时额外进行了10次448\*448的微调。经过微调后，V2 版本的 mAP 值提升了约4%。

## Convolutional



