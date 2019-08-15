## 摘要
本文的主要研究内容是通过实验，比较不同编程语言在serverless平台上运行的性能、成本、效率以及适合的应用。

## 介绍
intrinsically 本质上

本文主要是通过隔离function本身的性能影响，分析不同语言环境在容器中初始化的耗时。

主要用了AWS Lambda和Azure两个无服务平台以及一个SPF的无服务平台上的性能测试工具

AWS上比较了 .NET Core, Java, Python, NodeJS ,Go

AZure上比较了 C# 和 NodeJS

在cold start和warm start上都进行了比较

## serverless回顾
一个function启动时，会使用一个预先存在的容器为其配置环境依赖。

#### A. 无服务器性能注意事项
冷启动与热启动的环境不同

#### B. 无服务器成本注意事项
- 与传统云服务的对比
- 收费模型

## 无服务性能测试
AWS上五种语言，Azure上2种，交叉比较。

冷启动环境下的测量间隔设定为1h，热启动设定为1min，该实验仅仅是为了测试不同语言的环境初始化时间，因此function都为空的函数

#### 无服务性能框架

#### AWS测试结果
warm start
![](https://note.youdao.com/yws/public/resource/26ab61a1abef5caedf9b451b7ab61f35/xmlnote/5E177107612C4D76A14478BA7B6D9F66/17789)

cold start
![](https://note.youdao.com/yws/public/resource/26ab61a1abef5caedf9b451b7ab61f35/xmlnote/64F889F5F0F1419DBDDE86E407A4569E/17796)


- .Net和Java在冷启动时的耗时要远远超过热启动。**但是Python和Go冷启动的时间却低于热启动将近50%！**
- .Net适合于频繁使用的场景（热启动速度很快）

## Azure测试结果
warm start
- C# 0.93ms 
- NodeJS 4.91ms

cold start
- C# 16.84ms
- NodeJS 276.42ms




## AWS和AZure对比
基于NodeJS和C#


总的对比
![](https://note.youdao.com/yws/public/resource/26ab61a1abef5caedf9b451b7ab61f35/xmlnote/048B4677347740B1A52FB8E66D5D80A0/17823)

可以看到Azure种.Net的启动效率远远高于AWS，而AWS的NodeJS效率更高，这可能是因为Azure的容器是基于windows内核，而AWS是基于Linux内核。

## 成本分析

#### AWS计费
计费方式：
- $0.2每百万次
- $0.00001667 每 GB/s

AWS不同语言冷启动收费情况
![](https://note.youdao.com/yws/public/resource/26ab61a1abef5caedf9b451b7ab61f35/xmlnote/0F2E672F778246E29BD94B548335DEE5/17843)

这里不考虑热启动，因为收费是以100ms为单位，小于100ms，都按100ms计费，皆为0.41$

很明显JAVA和.Net由于初始化时间的问题大大影响到了计费成本。

#### Azure成本
和AWS不一样的是，Azure不能选择执行内存，所有function都是以128MB内存开始执行，然后动态分配内存

计费模式和AWS一致，结果如下：
![](https://note.youdao.com/yws/public/resource/26ab61a1abef5caedf9b451b7ab61f35/xmlnote/C6D631E045EF41B3B11C9C2D97C15E0D/17874)

## 结论
- Azure .Net效果最好
- AWS平台上最好的是Python
- 冷启动问题会严重影响成本









