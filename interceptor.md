# 拦截器

面向切面编程（AOP）方法可以让一个项目更加关注核心逻辑，常见的一些最佳实践包括

* 权限
* 缓存
* 错误处理
* 延时加载
* 调试
* 持久化
* 资源池
* 等等。。。

Rose拦截器实现AOP编程的一个手段，Rose将这些拦截器分为2种：局部拦截器和全局拦截器。

#### 局部拦截器

所谓局部拦截器是指，那些只能作用于某个模块的拦截器。当你把拦截器创建在controllers包或子包下，就意味它是一个局部拦截器；或者你把拦截器器创建在其他包下，但把它作为一个bean配置到controllers包或子包下的applicationContext.xml文件中，这时它也是一个局部拦截器。他被创建的地方或配置的地方称为“所在”的模块。特别的，对于通过配置形式的局部拦截器，其所在的模块可能有多个。

局部拦截器只能应用于“所在”的模块或其子模块，其他模块不会看到这个拦截器。需要特别强调的，rose所称“模块”和package是相关的，但不完全一致。不同的package一定属于不同模块，相同的package，但属于不同的jar包、不同的classes地址也是不同的模块。

局部拦截器默认可以应用于所在模块的子模块，但可以通过在该拦截器上标识@NotForSubModules禁止这种默认行为。

#### 全局拦截器

所谓全局拦截器是指，那些不是局部拦截器的其它拦截器，并且作为bean配置到root context中的拦截器。所谓root context默认是由
WEB-INF/applicationContext\*.xml、
classes/applicationContext\*.xml、
xxx.jar/applicationContext\*.xml
组成的ApplicationContext。

如果你的拦截器是公司级别的，希望在多个项目中共用的，就可以考虑提供一个单独的拦截器包，把拦截器创建在这个包中，并把它配置在src/applicationContext-intercetors.xml下。

为了使jar文件根目录下的applicationContext\*.xml能够被rose“认识到”，你需要在xxx.jar/META-INF下创建一个rose.properties文件，并写入一个属性rose=applicationContext 。如果没有这个属性，即使你的xxx.jar包根目录含有applicationContext\*.xml文件，也不会被rose识别。

#### 拦截器例子

```java
public class AccessTrackInterceptor extends ControllerInterceptorAdapter {
    public AccessTrackInterceptor() {
    	setPriority(29600);
    }
    @Override
    public Class<? extends Annotation> getRequiredAnnotationClass() {
        return PriCheckRequired.class; // 这是一个注解，只有标过的controller才会接受这个拦截器的洗礼。
    }
    @Override
    public Object before(Invocation inv) throws Exception {
        // TODO ....
    	return super.before(inv);
    }
    @Override
    public void afterCompletion(final Invocation inv, Throwable ex) throws Exception {
    	// TODO ....
    }
}
```

需要注意几点：

* 拦截器要放在controllers下(高级用法:打在rose-jar包里)
* 继承net.paoding.rose.web.ControllerInterceptorAdapter
* 按照实现的方法名，在controller执行前、中、后执行：
	* before：在controller执行前执行。
	* after：在controller执行中（后）执行，如果一个返回抛出了异常，则不会进来。
	* afterCompletion：在controller执行后执行，不论是否异常，都会进来。
	* isForAction：定义满足某条件的才会被拦截。

#### 拦截器可动的位置细节

上面都讲得差不多了，实际上还有不少地方可以拦截的：

* isForDispatcher：根据响应的情况判断是否拦截，比如说是正常请求、内部forward、还是include （但是没用过）
* setPriority：设置一个数字表示拦截优先级，当有多个拦截器时，要精准控制，数字小的内层，大的在外层，在最外层的before方法最先执行，大家都执行完后它的after才最后执行。
* round：这才是真正的controller执行中执行，不过用得很少。
* getRequiredAnnotationClass：返回一个Annotation class name，表示这个拦截器只对此Annotation标过的controller才生效。常用。

#### 实际应用场景

* 全站是否登录判断相关的逻辑，写在一个拦截器里，一次完成后，其他地方不再关心这个代码，在需要登录才能做的controller上注解一下，表示需要被执行拦截。
* 日志收集的逻辑，在一个拦截器里进行当前的access log记录。
* 权限体系的逻辑，写在一个拦截器里，在对应的操作上作注解，拦截器中进行细节的判断，新加的api也只是需要一次注解就得到了权限的判断。

### 参考
* [controller层：拦截器支持](http://www.54chen.com/java-ee/rose-3-2.html)
* [Rose_Code_Fragment_Interceptor](https://code.google.com/archive/p/paoding-rose/wikis/Rose_Code_Fragment_Interceptor.wiki)