## 1. Introduction

surge 一种上升的趋势

time consuming job 耗时的工作

我们测量用户看到的性能，以及在AWS lambda无服务器计算平台上运行三个经过MXnet训练过的深度学习模型的成本。

我们初步的一组实验结果表明，无服务器平台适合部署深度学习预测服务，但是由于运行第一个lambda调用而发生的延迟可能会扭曲延迟分布，从而有违反更严格的SLA的风险。


## 2. 背景

#### A. 无服务计算

冷启动现象导致额外的时间开销

#### B. 深度学习

从其根源到成为人工智能最新技术的深度学习的兴起，是由三个最新趋势推动的：培训数据量的激增、图形处理单元（GPU）等加速器的使用以及用于培训的模型设计的进步。

#### C. 实验

三个主流的图像识别模型：
- SqueezeNet  5MB 有理论上最高的识别率
- ResNet 18层 45MB
- ResNeXt-50 


为了避免下载造成的延迟，我们把图片当作依赖上传到了AWS Lambda

用API GATEWAY进行访问

计费：
![image](https://note.youdao.com/yws/public/resource/d1ad560dad03972fcc733df83bdef7d7/xmlnote/6FA9CB214C5E4DC28A279F68B7A39A6B/16107)

测试环境有两个：冷启动和热启动

度量指标：
- 响应时间
- 预测时间（模型输出结果的时间）
- 总的花费

#### A. 冷启动与热启动的评估
测试冷启动：
- 5次请求，每次隔十分钟

测试热启动：
- 第一次请求之后，隔一会儿连续发二十五个请求，每次间隔一秒

#### B. 热启动结果
以下分别是三个模型的实验结果

![image](https://note.youdao.com/yws/public/resource/d1ad560dad03972fcc733df83bdef7d7/xmlnote/77E08B72FED34929A9BEA7633418CAE5/16135)
![image](https://note.youdao.com/yws/public/resource/d1ad560dad03972fcc733df83bdef7d7/xmlnote/C7F4218A976343B7825705B23B0D6231/16137)
![image](https://note.youdao.com/yws/public/resource/d1ad560dad03972fcc733df83bdef7d7/xmlnote/3EF4AA69C4184F01A8B2DFE236ADA425/16139)

可以看出，预测时间与响应时间基本保持一致的变化趋势，这很容易想到，因为预测时间是响应时间的一部分。

另外，总成本和性能不一定会随着内存的上升而增加。

#### C. 冷启动
这里只放一个例子
![image](https://note.youdao.com/yws/public/resource/d1ad560dad03972fcc733df83bdef7d7/xmlnote/C8B018D687884272A69129A721D55D51/16159)

可以看到，即使内存调整到最大，仍然存在很大的延迟，同时预测时间和响应时间之差基本不变，说明容器初始化时间是一个常量。

#### D. 扩展性测试
图中显示，随着内存的增加，延迟和预测时间减少。该平台似乎随需求而扩展，尤其是对于大内存大小，其中延迟通常低于可接受的用户预期响应时间。

#### E. 讨论和限制

需要解决的问题：
- 冷启动耗时问题
- 总的延迟还是较高，可以考虑GPU资源的使用
- 云服务提供商最大资源的分配粒度不够，比如磁盘大小只给512，如果一个模型大于500怎么办


