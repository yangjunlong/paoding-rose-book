# paoding-rose

	人人网、糯米网释出的、开源的高效Java web开发框架。在小米米聊服务端再次被验证和使用。
	一个从零开始的创业公司，在大家技术背景不一的情况下，Rose很简单快速地传达到了大家中间。
	本手册致力于让php开发人员也能快速使用上java开发高性能服务。

* [前言](#前言)
* [Rose是什么](#是什么)
* [Rose能做什么](#能做什么)

### 前言
该文档主要记录学习`Paoding-Rose`框架的过程，因个人工作中时常会接触到基于该框架的一些项目，为了熟悉该框架以便于更好的与前端项目相融合，故整理此文档，其中大部分文档内容整理自官方文档。

为了便于说明，以下将`Paoding-Rose`简称为`Rose`

---

Rose是面向使用Java开发的同仁们的。Rose提供的各种特性和约束惯例，目的就是为了使您在能够轻松地开发web程序。如果您觉得Grails的想法很好，您不必转向它，Rose可以给您这种感觉，同时基于您对Java的熟悉，您又能更好地控制Rose。

我们希望Rose对各种技术的整合和规范，能使您摆脱犹豫，摆脱选择的困难，规避没有经验带来的开发风险。Rose不仅整合技术，同时还强调最佳实践，甚至包括名称规范。我们不仅仅只是提供技术，我们还会引导您应该如何使用好技术。

Rose规范了对Spring的使用，虽然大部分时间之内，您可能只是使用 @Autowired 即可，大多数时候的确这样也就够了。但 Rose 也允许您放置applicationContext-xxx.xml文件来扩展Rose。

不熟悉Spring的人，不用去重温Spring的知识也能够开始，并书写漂亮的程序！对熟悉Spring的人，则你们可以看到更多。

### 是什么?

* 基于IoC容器 (使用Spring 2.5.6).
* 收集最佳实践，形成规范和惯例，引导按规范惯例，简便开发.
* 收集通用功能，形成一些可使用的组件，提高生产效率.
* 特性的插拔，使用基于组合而非继承的设计.
* 提供可扩展的点，保持框架的可扩展性.
* 注重使用简易性的同时，注重内部代码设计和实现.

>如果你是一个创业公司在选择php还是java，同时如果你的团队有一个人写过一年java其他人都没写过。如果你想选择一个更加大型的系统框架，请使用rose，它收集了来自人人网、糯米网、小米科技的众多工程师的经验，你可以免费拥有这些。

### 能做什么?
1. **初级rose用户**
    1. rose可以用来完成一个网站。
2. **中级rose用户**
    1. rose可以用来完成一个大型网站，它提供的jade功能使得你的项目可以快速开发，自然切入连接池；它提供的portal功能，可以将一个网页分多个线程发起向DB的请求，节省用户的时间；它提供的pipe功能类似facebook的bigpipe，让前端加速，与此同时还有portal多线程的优势。
3. **高级rose用户**
    1. rose可以自由加入spring任何特性，比如定时执行（去TM的crontab）；比如拦截器做统一权限控制。可以自由配置主库从库，分表规则。配置thrift、zookeeper可以得到牛B的高可用性高性能服务集群。

### 参考
* [http://www.54chen.com/rose.html](http://www.54chen.com/rose.html)
* [http://code.google.com/p/paoding-rose/](http://code.google.com/p/paoding-rose/)
* [https://github.com/XiaoMi/rose](https://github.com/XiaoMi/rose)
* [入门指引](http://www.54chen.com/life/rose-manual-1.html)
* [Rose Philosophy](https://code.google.com/archive/p/paoding-rose/wikis/Rose_Philosophy.wiki)