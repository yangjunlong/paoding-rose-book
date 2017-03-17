# 原理概要

>Rose是在Spring基础上构建、符合Servlet规范、大量采用约定优于配置策略的WEB开发框架。

Rose框架通过在web.xml配置过滤器拦截并处理匹配的web请求，如果一个请求应该由在Rose框架下的类来处理， 该请求将在Rose调用中完成对客户端响应. 如果一个请求在Rose中没有找到合适的类来为他服务，Rose将把该请求移交给web容器的其他组件来处理。

Rose使用过滤器而非Servlet来接收web请求，这有它的合理性以及好处。

Servlet规范以“边走边看”的方式来处理请求，当服务器接收到一个web请求时，并没有要求在web.xml必须有相应的Servlet组件时才能处理，web请求被一系列Filter过滤时， Filter可以拿到相应的Request和Response对象 ，当Filter认为自己已经能够完成整个处理，它将不再调用chain.doNext()来使链中下个组件(Filter、Servlet、JSP)进行处理。

使用过滤器的好处是，Rose可以很好地和其他web框架兼容。这在改造遗留系统、对各种uri的支持具有天然优越性。正是使用过滤器，Rose不再要求请求地址具有特殊的后缀。

为了更好地理解，可以把Rose看成这样一种特殊的Servlet：它能够优先处理认定的事情，如无法处理再交给其它Filter、Servlet或JSP来处理。这个刚好是普通Servlet无法做到的 ： 如果一个请求以后缀名配置给他处理时候 ，一旦该Servlet处理不了，Servlet规范没有提供机制使得可以由配置在web.xml的其他正常组件处理 (除404，500等错误处理组件之外)。

一个web.xml中可能具有不只一个的Filter，Filter的先后顺序对系统具有重要影响，特别的，Rose自己的过滤器的配置顺序更是需要讲究 。如果一个请求在被Rose处理前，还应该被其它一些过滤器过滤，请把这些过滤器的mapping配置在Rose过滤器之前。

#### RoseFilter的配置
建议按以下配置即可：

```xml
<filter>
    <filter-name>roseFilter</filter-name>
    <filter-class>net.paoding.rose.RoseFilter</filter-class>
</filter>
<filter-mapping>
    <filter-name>roseFilter</filter-name>
    <url-pattern>/*</url-pattern>
    <dispatcher>REQUEST</dispatcher>
    <dispatcher>FORWARD</dispatcher>
    <dispatcher>INCLUDE</dispatcher>
</filter-mapping>
```

1. 大多数请况下，filter-mapping 应配置在所有Filter Mapping的最后。
2. 不能将 FORWARD、INCLUDE 的 dispatcher 去掉，否则forward、 include的请求Rose框架将拦截不到

Rose框架内部采用"匹配->执行"两阶段逻辑。Rose内部结构具有一个匹配树，这个数据结构可以快速判断一个请求是否应该由Rose处理并进行，没有找到匹配的请求交给过滤器的下一个组件处理。匹配成功的请求将进入”执行“阶段。 执行阶段需要经过6个步骤处理：`参数解析 -〉 验证器 -〉 拦截器 -〉 控制器 -〉 视图渲染 -〉渲染后`的处理链。

#### 匹配树
匹配树是一个多叉树，下面是一个例子：

```xml
ROOT
    GET="HomeController#index"
    package="com.xiaonei.xxx.controllers"
    /about
    GET="HomeController#about"
    package="com.xiaonei.xxx.controllers"
    /book
    GET="BookController#list"
    package="com.xiaonei.xxx.controllers.sub"
    POST="BookController#add" package="com.xiaonei.xxx.controllers.sub"
    /book/
    	/book/{id}
    		GET="BookController#show"
    		package="com.xiaonei.xxx.controllers.sub"
    		/help
    GET="HomeController#help"
    package="com.xiaonei.xxx.controllers"
```

ROOT代表这是一个根地址，也就是 http://localhost/ 代表的地址；

ROOT的下级有个GET结点，代表对该地址支持GET访问，不支持POST等其它访问，如果进行POST访问将以405错误回应。

/book代表这是一个/book地址，也就是 http://localhost/book 代表的地址；

/book下级有GET、POST两个结点，说明它支持GET和POST方法，根据HTTP语义，GET代表浏览，POST代表追加(向一个集合中追加一个条目)。

/book下还有/book/地址，这个地址有点特别，它以'/'结尾，但实际它不会被任何地址访问到，rose对http://localhost/book/ 的处理会将它等价于 http://localhost/book

这个特别的地址的存在完全是匹配树结构所需导致的，但不对实际匹配有任何坏的影响，所以也没有任何GET、POST等子结点。

/book/{id}代表是一个/book/123456、/book/654321这样的地址，当然这可以支持正则表达式的。

大部分情况下，匹配树的结构和实际的URI结构会一致，也因此匹配树的深度并不固定，每一个中间结点或叶子节点都有可能代表一个最终的URI地址，可以处理GET、POST等请求。对于那些匹配树存在的地址，但没有GET、POST、DELETE等子结点的，一旦用户请求了该地址，rose将直接把该请求转交给web容器处理，如果容器也不能处理它，最终用户将得到404响应。

#### 匹配过程
Rose以请求的地址作为处理输入(不包含Query串，即问号后的字符串)。如果匹配树中存在对应的地址，且含有对应请求方法(GET、POST、PUT、DELETE)的，则表示匹配成功；如果含有其他方法的，但没有当前方法的（比如只支持GET，但当前是POST的），则也表示匹配成功，但最后会以405响应出去；如果所给的地址没有任何支持的方法或者没有找到匹配地址的，则表示匹配失败。1.0.1不支持回朔算法，1.0.2将支持部分回朔算法(待发布时再做详细介绍)。

#### 参数解析
在调用验证器、拦截器 控制器之前，Rose完成2个解析：解析匹配树上动态的参数出实际值，解析控制器方法中参数实际的值。参数可能会解析失败(例如转化异常等等 )，此时该参数以默认值进行代替，同时Rose解析失败和异常记录起来放到专门的类中，继续下一个过程而不打断执行。

#### 拦截器
Rose使用自定义的拦截器接口而非一般的拦截器接口这是有理由的。使用Rose自定义的拦截器接口可以更容易地操作、控制Rose拦截。 所谓拦截即是对已经匹配的控制器调用进行拦截，在其调用之前、之后以及页面渲染之后执行某些逻辑。设计良好的拦截器可以被多个控制器使用。

#### 控制器

### Rose和Spring的关系
作为一名现代的Java开发者，几乎是离不开Spring的。Spring是SpringSource.org公司提供的开源Java基础架构产品。我们并不完全迷信Spring，但从技术上讲Spring提供了很好的在web开发中使用的基础组件、基础容器，使得它得到我们很大的尊重。

因为Spring充分地使用面向对象原则开发使Spring的丰富基础组件具有良好的使用和复用，有关web的一些基础组件包括：参数对象绑定器、Multipart对象抽象、视图渲染器等。这些组件被 Spring 自己的 Spring-MVC 框架自己使用，同时也被 Rose 大量使用。

除了对spring基础组件的使用，实际最重要的是Rose极大重视了对Spring ApplicationContext的使用。因为ApplicationContext的强大、聪慧，使得Rose的架构变得轻松而简单。每种框架对对象都有它的管理策略，Rose从本质上就是将各种各样的对象放置在合适的ApplicationContext上，不同的ApplicationContext之间形成上下级关系，让Spring来管理，并和开发者自定义的ApplicationContext有机结合起来。所以，从Spring的角度出发，开发者可以把Rose看出成是Spring的另外一种配置方式。也因此，Spring的@Autowired 可在Rose的类中大量被使用。

当然，为了更好地完成WEB工作以及极大的贯彻约定，不在没有意义的多种选择中彷徨，我们提供了Spring所没有的(或不同的)、Rose特有的各种机制：URI映射机制、参数映射机制以及各种道具的关系假设(拦截器、错误处理器、参数解析器)、页面映射关系、国际化配置机制等等。

###Rose与类反射

使用Rose进行WEB开发，第一个程序可能就是在某个controllers package下创建了某控制器类。Rose的控制器不需要什么接口或继承扩展什么类。同时控制器类的WEB动作方法几乎也是普通方法：其一：我们对方法名或方法参数没有强制规定；其二、一个控制器可以包括多少个WEB动作方法不做数量限制。

在这种框架下，虽然我们 有办法 通过在运行时自动进行代码增强包装成Command而不使用类反射来实现目的，虽然预见到放在我们口袋里的方案比直接使用类反射技术性能肯定有所提高，但目前我们仍没有这种做，Rose仍直接是使用类反射技术来访问开发者定义的方法。这样的决定来自我们的经验：目前类反射看起来并不影响到我们WEB应用的性能和响应速度，而且在现代Java程序大量使用类反射技术环境中，单纯“优化”Rose成效 预计 有限。当然，在必要时，我们不会放过可能的一点点性能提供。

所以，到现在为止，开发者定义的控制器和方法最终是由Rose通过method.invoke方法调用(对于代理对象的，使用Proxy来调用，少见)来处理web请求。这是唯一的在“服务”状态中必使用的类反射地方。

为了提高Rose的服务性能，Rose在启动的时候，已经把所有的控制器类和WEB动作方法进行了“解读”。在这个阶段，Rose会把所有URI映射规则以及尽可能做的事事先做好(包括对method方法名以及相关要素的缓存)，使Rose在服务状态时，不用再通过类反射来做和类、方法有关的识别和判断，直接就可以从缓存里获取method对象以及相关的其他要素进行处理，相信这已经是类反射API的极致追求。

另外一个使用类反射的地方是，在控制器声明Bean参数的(表单对象)，这将促使Rose使用类反射技术，把或请求对象中获取的参数进行转化动态设置到Bean的属性中。


### CHANGELOG

0.9.6

1. 如果@Param没有设置默认值时，@Param会有一个内置字符串来表示开发者没有定义，现在更改这个字符串的值，并使用JAVA_DEFAULT来表示这个值
	【对应用程序没有影响】

2. 去除ParameterBindingResult对rejectValue方法的覆盖，使其回归原始意思；如果出现绑定类型错误(TypeMismatch)，改addError(filedError)登记这个错误，而非使用rejectValue
	【对应用程序没有影响】

3. 构造控制器的方法参数时，如因非type mismatch导致的异常(比如构造函数异常)，这个错误不再作为error登记到bindingResult，而是rethrow异常出来
	【可能影响到应用程序，使原先可以“因为构造函数错误但还能运行”的服务变得不可服务】

4. 支持对Bean的Date、DateTime、TimeStamp属性进行绑定
	【可能影响应用程序，会对原先不能解析的date属性进行试图解析，把请求参数中的值解析到该参数】

5. 执行异常时在异常信息中加注Rose和Spring的版本号
	【对应用程序没有影响】

6. 如果有特定的请求方法rose本身不支持(比如PROPFIND)，原来是抛IllegalArgumentException，现改为405 Method Not Allowed
	【可能影响应用程序，原来抛出的请求方法不支持的异常，现在改为method allowed出去】

7. bugfix: 如果控制器目录下的rose.properties的module.path定义为一个未以'/'开始的串，且父package没有为空package或没有有效的rose类时，未能自动加上父亲路径的bug
	【可能影响程序：如果原先的程序定义了rose.properties，定义了module.path非以/开始的，但中间含有/字符的要特别认真校对】

	如果module.path以/开始，表示这是他的映射路径；如果不是以/开始的，表示需要加上它的父路径，举例如下:

	```
	package rose.properties的module.path 实际的
	module.path controllers.application app /app
	controllers.application.abc abc /app/abc
	controllers.application.edf /edf /edf
	controllers.application2 /app2 /app2
	controllers.application2.xyz xyz /app2/xyz
	```

8. bugfix: 解决参数解析互串的问，本质上PropertyEditor线程不安全，不能使用同一个propertyEditor对所有线程同时服务。
	【对应用程序没有影响；但因此造成的原来出现的一些“串页”问题将得到避免】

9. 支持ignoredPaths设置，使rose不对配置的地址进行映射和处理
配置方式：

	在web.xml中，roseFilter的param name为ignoredPaths，value是可逗号分隔的多项地址
	地址可有4种配置方式(均大小写敏感)：相等方式、前缀方式、后缀方式、正则表达式方式

10. 支持:continue指示，在控制器、拦截器、验证器等返回此字符串指示rose把这个请求送给下一个过滤器以及Servlet

11. 增加inv.getModel(String name)方法，等价于inv.getModel().get(String name)

12. 增加ParamValidator接口，和拦截器、控制器有类同语义的风格的处理方式

13. 修改model.getAttributes语义，不再返回rose框架自身设置的model数据(即keyt以$$paoding-rose开始的属性)

### 参考
* [Rose原理概要](https://code.google.com/archive/p/paoding-rose/wikis/Rose_Architect.wiki)