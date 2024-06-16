#    

可以说，YOLO各代升级的改进点都是提升检测效果mAP和速度FPS。 

## Precision精度与Recall召回率

要计算精度与召回率（查全率），我们需要先了解四个值：

- TP（True Positives）
- FP（False Positives）
- FN（False Negatives）
- TN（True Negatives）

|                                 | 相关（Relevant），正类 | 无关（NonRelevant），负类 |
| :------------------------------ | :--------------------- | ------------------------- |
| **被检索到（Retrieved）**       | TP，正类判定为正类。   | FP，负类判定为正类        |
| **未被检索到（Not Retrieved）** | FN，正类判定为负类     | TN，负类判断为负类        |

### 记忆方法

我们只需要记住这里面的四个单词的中文意思方可推断出四个值的含义。True 正确的、False 错误的、Positives 正类以及 Negatives 负类。

- True Positives（:heavy_plus_sign::heavy_plus_sign:）——正确的判断为正类，即将正类判定为正类。
- False Positives（:heavy_minus_sign::heavy_plus_sign:）——错误的判定为正类，即将负类判定为正类。
- False Negatives（:heavy_plus_sign::heavy_minus_sign:）——错误的判定为负类，即将正类判定为正类。

- True Negatives（:heavy_minus_sign::heavy_minus_sign:）——正确的判定为负类，即将负类判定为负类。

### 计算公式

$$
Precision = \frac{TP}{TP+FP}
$$

$$
Recall = \frac{TP}{TP + FN}
$$

![IMG_4784](https://s2.loli.net/2023/08/28/DLUk8I91clvCbwd.jpg)

为了方便理解，我画了一个草图，以格子为单位。其中蓝色框代表实际值，橙色框代表预测值。

真正的正类（蓝色框内）共20个格子，真正的负类（蓝色框外）共10个格子。判定的正类（橙色框内），判定的负类（橙色框外）

TP值（正确的判断为正类），真正的正类与判定的正类的交集，即涂黄色的格子，共9个格子。

FP值（错误的判定为正类），真正的负类与判定的正类的交集，即涂绿色的格子，共6个格子。

FN值（错误的判定为负类），真正的正类与判定的负类的交集，即涂蓝色的格子，共11个格子。

TN值（正确的判定为负类），真正的负类与判定的负类的交集，即涂红色的格子，共4个格子。

### 例子

已知条件：班级总人数100人，其中男生80人，女生20人。
目标：找出所有的女生。
结果：从班级中选择了50人，其中20人是女生，还错误的把30名男生挑选出来了。

TP = 20; FP = 30; FN = 0; TN = 50

## mAP指标

目标检测不可单看精度（Precision）或召回率（Recall），因为两者为此消彼长的关系。所以需要一个新的指标用于综合的衡量目标检测的效果。

mAP（mean Average Precision）指标，又名全类平均精度，用于综合衡量目标检测的效果。

它是多种类别的AP值的平均值。关于AP值如何计算将在下面介绍。

## IoU指标

IoU（Intersection of unit）意思就是交集和并集的比值。在目标检测中，即真实值（蓝色框）和预测值（橙色框）的交集和其并集的比值。

![IoU指标](https://s2.loli.net/2023/08/28/UKzvVBHDgPc8jkq.png)

IoU值越高表示预测值和真实值越吻合，即检测越准确。

## 检测任务中的检测指标

### 置信度

置信度(Confidence)是模型对检测出的每个目标框预测正确的概率或确信程度。

例如下面三图中，绿色框左上角的红色数字0.9、0.8和0.7都代表这个框内物体为人脸的概率。

![检测任务中的精度和召回率](/Users/hochihchou/Library/Application Support/CleanShot/media/media_IktPRFVgrL/CleanShot 2023-08-29 at 13.43.01@2x.png)

### 计算精度和召回率

在一次检测任务中，一般基于置信度阈值来计算此次检测任务的精度和召回率。

在一次检测中，算法会产生很多的检测框，框的置信度不一，因此需要设置一个置信度阈值来过滤掉置信度较低的框，只留下置信度高的检测框。

还是上图，当置信度阈值为0.9时，只有左图置信度大于等于0.9，因此TP = 1、FP = 0、FN = 2；

计算可得精度 Precision = 1/1； 召回率Recall = 1/3。

### 计算AP值与mAP值

计算mAP值之前需要先计算某一类别的AP值，如人脸检测的AP值。将精度Precision和召回率Recall分别放在直角坐标系的Y轴和X轴，这样就得到了PR图（Precision x Recall curve）。

![CleanShot 2023-08-29 at 13.54.18@2x](https://s2.loli.net/2023/08/29/DN4uysQH3dhXGT8.png)

左图中蓝实线 Precision 为原始精度，为锯齿状，不符合一般化规律。
红虚线 Interpolated precision 为插值后的精度，插值方法简单来说就是将某点的精确率替换为该召回率右侧的所有召回率对应的精确率的最大值。

如右图所示，AP值即为红虚线（插值后的精度）下方的面积。

计算出各个种类（人脸、苹果、骑车）的AP值后，再求它们的平均值即为mAP值。
