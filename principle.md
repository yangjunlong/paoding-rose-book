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
CHANGELOG

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
* https://code.google.com/archive/p/paoding-rose/