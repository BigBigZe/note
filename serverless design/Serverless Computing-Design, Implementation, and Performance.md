## 1.Introduction
无服务计算是基于事件的

在物联网技术中，可以被广泛使用

## 2. 原型设计
采用的云平台是Microsoft Azure，持久层为 Azure Storage ，同时提供了一个Web接口以及一个worker服务（管理执行function）

web服务通过AS中的消息层来进行worker的发现

function的元数据被存放在AS table中，代码被存放在AS blob中。

如图为其架构图:

![image](https://note.youdao.com/yws/public/resource/c601f52e6d5d27c68644a25707da2ba4/xmlnote/AB55971BA5BF4317820B2009B6F0F651/15781)

选择AS作为持久层，是因为AS 通过一个简单的API来提供了高度可扩展和低延迟的服务。

#### A. Function Metadata
一个function与很多实体相关联，包括它的元数据、代码、运行容器以及“热队列”。

function metadata由以下四个部分构成：
- function 标识符。随机生成一个GUID，用于标识function资源。
- 运行时语言。function所采用的语言，本文的serverless平台采用Node.js
- 内存大小。maximum memory
- 代码块URI。函数创建过程中提供了包含函数代码的zip存档。此代码被复制到平台存储帐户中的一个blob，该blob的uri被放置在函数的元数据中。

#### B. Function Execution
我们的实现只提供了一种简单的编程模型，只提供手动的调用。

function通过REST API “/invoke”进行调用

调用的请求主体包含了function的输入，而响应主题包含了function的输出。

一个请求对象包含了所有的function metadata，然后web service找到一个能用的container处理请求。

web和worker之间的交互是通过一个共享的消息层实现的，而每个function 都有"cold queue"和"warm queue"两个队列，这些队列存的是可获取容器的信息。


冷队列中的消息表示工作线程有未分配的内存，可以在其中启动容器，而函数的热队列中的可见消息则表示当前未处理执行请求的现有函数容器。


即冷队列中消息表示闲置可用内存；热队列消息表示已经初始化的能够使用的function实体

请求先从热队列拿消息，没拿到就从冷队列拿，然后新建一个container给worker。如果两个队列都没拿到，说明内存耗尽。  

一个web service如果拿到了message，就会根据里面的URI给相应的worker发送请求。

#### C. 容器分配

每个worker 都有一个未分配的资源池。

一旦一块内存分配给一个容器，那么就会有一个唯一标识符和URI给存到消息队列。

当message是被放在冷队列时，说明该容器内没有function实例

为了确保工作人员不会过度配置其内存池，假定分配的函数将具有最大的函数内存大小。然后，当一个工作服务接收到一个未分配分配的执行请求时，如果分配的函数要求的大小小于最大值，它将回收内存。在创建容器并首次执行其函数后，容器分配消息将放置在该函数的热队列中。

#### D. 容器移除

移除一个容器有两种方式。
- 首先，当一个函数被删除时，Web服务将删除该函数的热队列，该热队列由保存该函数容器的worker实例定期监视是否存在。如果worker检测到一个已删除的函数队列，它将删除该函数正在运行的容器并回收它们的内存保留。
- 如果一个container在一个周期内idle时间过长，也会被自动清除，这个周期可自己设定。



implications 启示

expiration  有效期

#### E. 容器镜像


镜像包括function运行时间以及运行处理函数

自定义容器不是为平台中的每个函数构建的，而是在启动容器时附加一个包含函数代码的只读卷。


我们用的是Windows Nano Server 类型的Image环境，而不是传统的Linux。

## 3. 性能测试结果
开发了一个性能测试工具，使用了一个运行时间很短的function作为测试用例。

除了Azure，其余平台都是设为512MB的最大运行内存（Azure会自动检测分配）

#### A.并发性测试








