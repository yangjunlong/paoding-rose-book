# ErrorHandler支持

#### ErrorHandler的作用
一般来说传统的编程都会到处去try，特别是java里，try来try去的（如果你用erlang一定就知道，已经知道的可能性，怎么能叫异常？都try了还是让它崩了算了。。。）。

如果打开你的项目，每个java文件中的代码都有一堆的try，那这时候就是ErrorHandle上阵的时候了。

ErrorHanle致力于：统一捕捉和处理各种异常，可区分对待和返回；统一的出错体验。

非常类似做web开发时的500统一出错页面这样的东东。

#### 示例

```java
package com.xiaonei.rose.usage.controllers;

import net.paoding.rose.web.ControllerErrorHandler;
import net.paoding.rose.web.Invocation;

public class ErrorHandler implements ControllerErrorHandler {
    public Object onError(Invocation inv, Throwable ex) throws Throwable {
        // TODO logger.error("handle err:", ex);
        return "@error";
    }
}
```

#### 放在哪里才能生效？

* 放在controllers目录下，和controller们在一起（幸福快乐地生活）。
* 一般来讲，ErrorHandler都是用在web项目里，在最外层起作用。
* 所有的方法都可以尽情地向throws Exception了。
* 不需要再try。

#### 记录错误日志 & 使用专门的错误页面提示用户
```java
// 直接放在controllers或其子package下(也可以放到其他package下，但需要applicationContext配置)

package com.xiaonei.rose.usage.controllers;
import net.paoding.rose.web.ControllerErrorHandler;
import net.paoding.rose.web.Invocation;

public class ErrorHandler implements ControllerErrorHandler {
    public Object onError(Invocation inv, Throwable ex) {
        Log logger = LogFactory.getLog(inv.getControllerClass());
        logger.error("", ex);
        // forward to webapp/views/500.jsp
        return "/views/500.jsp";
    }
}

```

#### 通过code提示不同的错误信息

```java
import net.paoding.rose.web.ControllerErrorHandler;

public class ErrorHandler implements ControllerErrorHandler {
    @Override
    public Object onError(Invocation inv, Throwable ex) {
        if (ex instanceof BizException) {
            BizException bizEx = (BizException) ex;
            String code = bizEx.getCode();
            // 在控制器所在的package中或WEB-INF目录下配置messages.xml，
            // 可配置多个，优先找控制器自己package的，然后是父package的，最后是WEB-INF的
            // messages.xml格式参考在下面
            MessageSource msgSource = inv.getApplicationContext();
            String msg = msgSource.getMessage(code, bizEx.getArgs(), inv.getRequest().getLocale());
            // 在jsp中使用${errorMsg}输出该错误
            inv.addModel("errorMsg", msg); 
            return "/views/biz-500.jsp";
        }

        Log logger = LogFactory.getLog(inv.getControllerClass());
        logger.error("", ex);
        // forward to webapp/views/500.jsp
        return "/views/500.jsp";
    }
}
```

**messages.xml格式参考**
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE properties SYSTEM "http://java.sun.com/dtd/properties.dtd">
<properties>
	<comment>Rhyme</comment>
	<entry key="seven-eight">lay them straight</entry>
	<entry key="five-six">pick up sticks</entry>
	<entry key="nine-ten">a big, fat hen</entry>
	<entry key="three-four">shut the door</entry>
	<entry key="one-two">buckle my shoe</entry>
</properties>
```

#### 更好的区分
```java
import net.paoding.rose.web.ControllerErrorHandler;

public class ErrorHandler implements ControllerErrorHandler {
    // @since 0.9.4 
    // 把方法第2个参数换上具体的异常类...这个onError就只处理所声明的这类异常
    public Object onError(Invocation inv, BizException bizEx) {
        // 略去具体的处理代码......
        return "/views/biz-500.jsp";
    }

    // 通用onError方法，处理其他onError无法处理的异常
    @Override
    public Object onError(Invocation inv, Throwable ex) {
        // 略去具体的处理代码......
        return "/views/500.jsp";
    }
}
```

#### 将异常让渡给上级模块的错误处理器处理
1. controllers自己或其子package下都可以拥有独立的ControllerErrorHandler。
如果在web调用过程中，控制器、拦截器等发生异常时，如果给定的module含有自己ControllerErrorHanlder时，则由他处理；如果自己没有则调用上级的ControllerErrorHandler处理。

2. 但是如果所在的module有ControllerErrorHandler，如何在有必要的时候将异常抛给上级的ControllerErrorHandler呢？

```java
package com.xiaonei.rose.usage.controllers.error;

import net.paoding.rose.web.ControllerErrorHandler;
import net.paoding.rose.web.Invocation;
import net.paoding.rose.web.ParentErrorHandler;
import org.springframework.beans.factory.annotation.Autowired;

public class ErrorHandler implements ControllerErrorHandler {
    // 声明ParentErrorHandler,注意，这里不是ControllerErrorHandler
    // 万一上级没有ControllerErrorHandler, 这个字段也不会为空
    @Autowired
    ParentErrorHandler parent;
    // 处理这个处理器只想处理的
    public Object onError(Invocation inv, RuntimeException ex) throws Throwable {
        System.out.println("---------RuntimeException----------");
        inv.getResponse().getWriter().write("<pre>RuntimeException<br>");
        ex.printStackTrace(inv.getResponse().getWriter());
        inv.getResponse().getWriter().write("</pre>");
        return "";
    }

    // 通用的异常抛给上级ControllerErrorHanlder或上级的上级去处理
    @Override
    public Object onError(Invocation inv, Throwable ex) throws Throwable {
        return parent.onError(inv, ex);
    }
}

```
以上parentErrorHanlder的逻辑，Rose提供的ErrorHandlerAdpater类已经封装了
建议您通过extends ErrorHandlerAdapter 实现错误处理器，而非直接实现ControllerErrorHandler

* 处理这个处理器只想处理的
* 通用的默认就会抛给上级模块的错误处理器处理

```java
public class ErrorHandler extends ErrorHandlerAdapter {
    public Object onError(Invocation inv, RuntimeException ex) throws Throwable {
        System.out.println("---------RuntimeException----------");
        inv.getResponse().getWriter().write("<pre>RuntimeException<br>");
        ex.printStackTrace(inv.getResponse().getWriter());
        inv.getResponse().getWriter().write("</pre>");
        return "";
    }
}
```

### 参考
* [Rose_Code_Fragment_ErrorHandler](https://code.google.com/archive/p/paoding-rose/wikis/Rose_Code_Fragment_ErrorHandler.wiki)
* [controller层：ErrorHandler支持](http://www.54chen.com/java-ee/rose-3-3.html)