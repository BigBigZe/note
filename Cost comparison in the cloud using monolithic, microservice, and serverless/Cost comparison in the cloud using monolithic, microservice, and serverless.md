## 摘要
微服务架构如今被全球的各大互联网厂商广泛应用于各种应用架构。除了微服务带给我们的灵活的扩展性、高内聚低耦合的特性以外，我们也关心在微服务本身的架构成本。

本文用一个web应用来对比以下三个场景下的架构成本：
- 传统的单体架构
- 基于云服务的微服务架构
- 基于serverless的微服务架构

实验结果表明基于AWS Lambda的server less平台去搭建微服务架构相比于传统的单体架构可以节省将近70%的成本。

## 简介
传统的云服务（IaaS、PaaS、SaaS）在实现一些特殊的分布式特性（自动扩展、热部署、高可靠性）时，会产生不同程度的时间与金钱成本。

单体服务：应用使用一个“代码库”将所有服务通过各种API暴露出去，所有的开发人员都可以对这个“代码库”进行修改。

单体应用的扩展是很困难的，因为一个应用中存在各种各样的消费模式（消费模式意味着服务之间的交互），所以需要一种新的架构能够不考虑服务之间是如何消费的。

由于所有的服务都在一个应用中存在，同时运行，因此可能那些不常使用的服务也要持续消耗资源，从而产生很多额外的费用。

be tailored to 量身定做

微服务通过将不同的服务解耦，使得每个服务都享有高度的自治权力，因此可以节省很多上述的费用。

然而，如果单纯的独立部署、扩展每一个服务，也是一件麻烦的事

AWS Lambda可以让用户不在关心服务器即所谓的分布式特性实现，并且按使用计费，大大节省使用成本。

## 背景
目前大部分主流应用还是部署在IaaS和PaaS 为主的服务平台上。


在单体应用中，假定：
- S:service
- D:developer
- O:operator

当需求增加时，S和D显然也会相应的增加，因此应用复杂性以及部署时间也会随之增加。

在大型商业应用中主流的解决方法是SOA，将一个大型商业应用分为多个应用，应用之间通过协议进行通信。SOA的主要策略是一个应用对应一个Team。

虽然SOA的确算一个不错的解决方案，但是时间和金钱成本以及复杂性都相对较高。

SOA的瓶颈主要在他使用了一个ESB总线进行服务之间的通信。对普通的企业应用的负载而言还可以适应，但是对于大型的互联网应用，要面对上亿的访问量显然不行。

微服务在避免了单体架构的问题的同时也继承了SOA很多优点。

微服务相较于SOA划分服务的粒度更加细。同时微服务通过网关将不同的服务API暴露给其他服务或不同的用户。

服务是暴露给网关的，而不是直接暴露给用户。

微服务存在的一个问题就是自动扩展和部署太麻烦了（几十或者几百个服务都需要部署和扩展）

为了解决这一问题，AWS Lambda被提出，用户不需要对服务器做任何管理，同时按使用付费。一旦服务被部署，就可以自动扩展。

## 用例

该应用程序旨在支持为机构向客户提供的贷款生成和查询付款计划的业务流程。

应用被提供给不同的租户，采用的是SaaS模式，同时每个租户（企业）会有一个管理员来管理该企业的所有用户，为普通用户分配访问权限（查询和制作账单）。

在数据库层，采用的是多租户共享数据库。

为了方便研究，值考虑两个服务：
- S1，生成支付计划的服务，包含一个个付款组（1-180个月）。CPU密集型，不需要在数据库存东西，响应时间大约3000ms
- S2，返回一个已存在的付款计划以及相关联的付款计划集合。具体过程是给一个ID，然后从数据库查询，相应时间大约300ms，对数据库需要进行频繁的查询操作。

#### 1. 单体架构
![image](https://note.youdao.com/yws/public/resource/26ab61a1abef5caedf9b451b7ab61f35/xmlnote/E8BA9C5FCCBC43E1A839AB57E70BDD04/17060)

单体架构用的是常见的MVC三层架构模式。

两个服务都是REST风格，可以同时部署到同一个服务器上去。同时为了能够增强其服务提供能力，采用了下图所示的拓展架构：

![image](https://note.youdao.com/yws/public/resource/26ab61a1abef5caedf9b451b7ab61f35/xmlnote/0752914307C24F1FB00A8DF3EAE9D3B2/17073)

使用一个服务器集群（多个服务器，每个服务器上都跑一个该应用）做负载均衡，同时使用一个共享多租户数据库。

#### 2. 由云服务使用者操作的微服务架构（搭建在传统PaaS上的微服务）

采用微服务架构的关键就在与如何划分出各个“微服务”，为了方便说明，以ms1和ms2两个微服务为例子进行阐述。

![image](https://note.youdao.com/yws/public/resource/26ab61a1abef5caedf9b451b7ab61f35/xmlnote/53F9CB862EBE4E9ABD9AA6BCAE6C4C87/17090)

可以看到，每个ms都相当于一个独立的应用，不同的颜色意味着不同的技术栈。

在交互方面，每个ms通过内网将自己的REST接口暴露给网关。由于ms1不需要持久层，所以不需要进行数据库关联。

网关作为一个简单的web应用接收browser的请求。

然后加上可扩展性以及负载均衡等特性，可以形成新的如下图所示的分布式架构模式：
![image](https://note.youdao.com/yws/public/resource/26ab61a1abef5caedf9b451b7ab61f35/xmlnote/EEB22A7824C3403993D8A802527EEF9A/17109)

可以看到，每个微服务（网关也算一个微服务）都可以独立的扩展

#### 3. 通过AWS Lambda来管理的微服务架构（基于FaaS的微服务架构）
在serverless架构中，最基本的单元是function，对应于微服务，就需要将ms转化为提供REST服务的function，包括gateway，也需要做成一个function

![image](https://note.youdao.com/yws/public/resource/26ab61a1abef5caedf9b451b7ab61f35/xmlnote/1DA838F4F68544DD9412405AF31609E8/17128)

不同的function之间通过JSON数据格式进行交互，web Client为前端的UI界面，用户第一次访问需要下载静态资源（HTML、JS等）

-----
该实验项目由一个团队完成，而实际上在单体架构中应该由两个团队负责：前端+后台；在serverless服务平台上则有四个：网关+ms1+ms2+前端

## 实现


传统架构采用两种技术栈：
- Play web framework with JAVA
- Jax-RS

特点是轻量、无状态、便于在分布式环境上架构

服务器用的是内嵌的Netty和Jetty，可以在几秒内启动

传统架构为基准组

Given that 基于此

#### 1. 单体架构
![image](https://note.youdao.com/yws/public/resource/26ab61a1abef5caedf9b451b7ab61f35/xmlnote/6FCE91325FD241ED87059D7381DD2375/17331)

#### 2. 基于IaaS的微服务架构
ms1、ms2、gateway采用的和单体架构种的web application一样的技术；前端对应于单体中的前端技术

#### 3. AWS Lambda
![image](https://note.youdao.com/yws/public/resource/26ab61a1abef5caedf9b451b7ab61f35/xmlnote/9579EA57A94F4EBBAC6043856FDF4632/17348)

## 部署
以单体架构为参照组，分别对三组不同的架构进行性能测试（测试能同时处理多少请求），其中对单体架构分别测试两种技术栈的性能。

#### 1. 单体架构部署
![image](https://note.youdao.com/yws/public/resource/26ab61a1abef5caedf9b451b7ab61f35/xmlnote/A84751C2171E4201B37FD0CAA86B130C/17360)

- 后台：4个large VM(2CPU 3.75GB)
- 数据库：1个medium VM(1CPU 3.75GB)

#### 2. 微服务架构
![image](https://note.youdao.com/yws/public/resource/26ab61a1abef5caedf9b451b7ab61f35/xmlnote/A47A88679F2D4B81B4945B8F0F301558/17369)

- ms1：3个large VM(2CPU 3.75GB)
- ms2: 1个small VM(1CPU 2GB)
- gateway：1个medium VM(1CPU 3.75GB)
- 数据库：1个medium VM(1CPU 3.75GB)

#### 3. 无服务架构的微服务架构
![image](https://note.youdao.com/yws/public/resource/26ab61a1abef5caedf9b451b7ab61f35/xmlnote/6C5467E106204C8EBE89FE7F14A74108/17403)

- ms1: 512MB
- ms2: 320MB
- database: medium VM
- gatewayS1: 512MB
- gatewayS2: 320MB
- 前端：静态资源放在s3

## 测试结果

#### 1. 性能测试
模拟三种场景：
- 20%请求s1；80%请求s2
- 50%请求s1；50%请求s2
- 20%请求s1；80%请求s2

单体架构模式的收费情况：
![image](https://note.youdao.com/yws/public/resource/26ab61a1abef5caedf9b451b7ab61f35/xmlnote/5287C260B0BC4B1D975E9140DF338E89/17429)

单体架构性能测试结果如下：
![image](https://note.youdao.com/yws/public/resource/26ab61a1abef5caedf9b451b7ab61f35/xmlnote/42F94E6DE8894DE5A2D3CD024054573D/17435)

IaaS的微服务架构收费情况：
![image](https://note.youdao.com/yws/public/resource/26ab61a1abef5caedf9b451b7ab61f35/xmlnote/DEFB28A574CE4BB2B10DDB480523B758/17443)

IaaS的微服务架构性能测试结果如下：
![image](https://note.youdao.com/yws/public/resource/26ab61a1abef5caedf9b451b7ab61f35/xmlnote/E98CA57ADA38410FA79B4FD7AFD9A9F4/17448)

AWS Lambda收费情况：
![image](https://note.youdao.com/yws/public/resource/26ab61a1abef5caedf9b451b7ab61f35/xmlnote/6ED192F3C3E04C53A66A87A2B490597E/17454)

AWS Lambda的请求数模拟的是和微服务架构相同的次数（因为AWS Lambda的请求理论上是没有上限的，这里为了和常规的微服务架构进行对比）


三种架构的费用对比（每百万条请求）：
![image](https://note.youdao.com/yws/public/resource/26ab61a1abef5caedf9b451b7ab61f35/xmlnote/FD21B0B56A7345B1813F42109C2B5127/17464)

#### 2. 响应时间
![image](https://note.youdao.com/yws/public/resource/26ab61a1abef5caedf9b451b7ab61f35/xmlnote/61B24C330350458EA8AF5CF8E11AD201/17473)

可以看到在普通的微服务架构中，响应时间较长，因为请求是需要经过网关处理的；而在serverless服务上的微服务架构中虽然也有网关，但是响应时间仍然很短，这是因为每次请求都独立在一个function实例中完成的，所以速度很快，而另外两种架构都需要在一个VM上同时处理很多请求。

#### 开发方法学
微服务架构中每个团队只需要专注自己的业务，并且技术隔离，但是部署和测试过程很麻烦；单体架构则需要更加专注于解决一些分布式特性问题（一致性、自动扩展等）；serverless开发在function的设计上需要花费一定时间，因为必须保证每个function的隔离性

#### 部署、扩展和持续交付
AWS-Lambda相当于一种ready-to-use的服务，因此对于微服务架构来说，只需要上传了代码就直接可以运行；但是如果不采用serverless这种服务，就需要手动进行很多配置。

## 讨论
future work：评测不同类型的服务——cpu密集型、IO密集型、网络传输密集型

