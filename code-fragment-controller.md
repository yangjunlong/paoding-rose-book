# 控制器代码书写示例片段
>本页面主要展示一些常用的控制器类代码片段，以期能够通过这些案例了解控制器的编写要领

* [起步](#起步)
* [返回一个Velocity页面](#返回一个velocity页面)
* [返回一个JSP页面](#返回一个jsp页面)
* [在页面渲染业务数据](#在页面渲染业务数据)
* [更改控制器映射](#更改控制器映射)
* [重定向(Redirect)](#重定向redirect)
* [转发(Forward)](#转发forward)
* [自定义方法映射](#自定义方法映射)
* [使用正则表达式自定义方法映射](#使用正则表达式自定义方法映射)
* [获取request请求参数](#获取request请求参数)
* [数组参数](#数组参数)
* [Map参数](#map参数)
* [表单提交](#表单提交)
* [设置内嵌对象的属性值](#设置内嵌对象的属性值)
* [返回JSON](#返回json)
* [返回XML](#返回xml)

### 起步
http://127.0.0.1/user/test

```java
// 控制器必须声明在controllers或其子package下，注意controllers后面有个s 
package com.xiaonei.rose.usage.controllers;

// 控制器要以Controller结尾，可不继承或实现其他类或接口 
public class UserController {
	public String test() {
		// 返回@开始的字符串，表示将紧跟@之后的字符串显示在页面上 
		return "@" + new java.util.Date(); 
	}
}
```

### 返回一个Velocity页面
http://127.0.0.1/user/velocity

```java
public class UserController {
	public String velocity() { 
		// 返回一个普通字符串，表示要从webapp/views/目录下找第一个以user-velocity.开始的页面 
		// 运行本程序时，请在webapp/views/目录下创建一个名为user-velocity.vm的文件，写上文本字符
		return "user-velocity";
	}
}
```

### 返回一个JSP页面
http://127.0.0.1/user/jsp

```java
public class UserController {
	public String jsp() {
	// 在webapp/views/目录下创建user-jsp.jsp的文件即可 (UTF-8的)。
	return "user-jsp"; 
	}
}
```

### 在页面渲染业务数据
http://127.0.0.1/user/render

```java
import net.paoding.rose.web.Invocation;

public class UserController {
	public String render(Invocation inv) { 
		// 在vm/jsp中可以使用$now渲染这个值 
		inv.addModel("now", new java.util.Date()); 

		// 在vm/jsp中可以使用$user.id, $user.name渲染user的值
		inv.addModel("user", new User(1, "qieqie.wang"));

		// id=1, name=qieqie.wang 
		return "user-render"; 
	}
}
```

### 更改控制器映射
http://127.0.0.1/u/test

```java
import net.paoding.rose.web.annotation.Path;

// 在控制器上标注@Path设置"u"，自定义映射规则(默认是/user，现改为/u) 
@Path("u") 
public class UserController {
	public String test() {
		return "@" + new java.util.Date(); 
	}
}
```

### 重定向(Redirect)
http://127.0.0.1/user/redirect

```java
public class UserController {
	public String redirect() {
		// 以r:开始表示重定向 
		return "r:/user/test";
		// 或 r:http://127.0.0.1/user/test 
	}
}
```

### 转发(Forward)

http://127.0.0.1/user/forward 

http://127.0.0.1/user/forward2

```java
public class UserController {
	public String forward() {
		// 大多数情况下，以/开始即是转发(除非存在webapp/user/test文件)
		return "/user/test";
	}

	public String forward2() {
		// a:开始表示转发到同一个控制器的acton方法forward()，更多参数有m:/module:,c:/controller:
		return "a:forward?note=可以带参数";
	}
}
```

### 自定义方法映射
http://127.0.0.1/user/list-by-group?groupId=123

```java
public class UserController {
	@Get("list-by-group")
	public String listByGroup(@Param("groupId") String groupId) {
		return "@${groupId}"; 
	}
}
```

### 使用正则表达式自定义方法映射
http://127.0.0.1/user/list-by-group-abc

```java
public class UserController {
	@Get("list-by-group-{groupId}") 
	public String listByGroup2(@Param("groupId") int groupId) { 
		return "@string-${groupId}";
	}
}
```

http://127.0.0.1/user/list-by-group-123

```java
public class UserController {
	@Get("list-by-group-{groupId:\\d+}") 
	public String listByGroup3(@Param("groupId") String groupId) {
		return "@int-${groupId}";
	}
}
```

有问题？访问 http://127.0.0.1/user/list-by-group-123 打印出"string-123"而非“int-123”?
这是因为listByGroup2和listByGroup3的path定义的"非常一样"，Rose自身无法判断哪个优先级更高(对不起，我们还没找到给正则表达式排序的有效方法)。

在此建议如下：

* 调整path定义：把第二个path改为list-by-group-n{groupId:\\d+}，然后通过http://127.0.0.1/user/list-by-group-n123 访问它
* list-by-group-n{groupId} 的优先级高于 list-by-group-{gourpId}，Rose先“问”前者，只有前者不能处理时(abc不是数字，所以其不能处理)，才走后者
* 想知道Rose的判断优先顺序？OK，请访问 http://localhost/rose-info/tree
* 当然，为安全考虑，rose-info/tree这个地址的可访问性，需要您明确把net.paoding.rose.web.controllers.roseInfo.TreeController的DEBUG log级别打开(通过log级别控制权限，算是我们特有的一种思路)

### 获取request请求参数
http://127.0.0.1/user/param1?name=rose 

```java
public String param1(@Param("name") String name) {
	return "@" + name;
}
```

http://127.0.0.1/user/param2?name=rose

```java
public String param2(Invocation inv) {
	return "@" + inv.getRequest().getParameter("name");
}
```

http://127.0.0.1/user/param3/rose

```java
@Get("param3/{name}")
public String param3(Invocation inv, @Param("name") String name) {
	// request.getParameter()也能获取@ReqMapping中定义的参数
	return "@method.name=" + name + "; request.param.name=" + inv.getRequest().getParameter("name");
}
```

### 数组参数
http://127.0.0.1/usre/array?id=1&id=2&id=3

http://127.0.0.1/usre/array?id=1,2,3,4

```java
public String array(@Param("id") int[] idArray) {
	return "@" + Arrays.toString(idArray);
}
```

### Map参数
http://127.0.0.1/user/keyOfMap?map:1=paoding&map:2=rose 

```java
public String keyOfMap(@Param("map") Map<Integer, String> map) {
	return "@" + Arrays.toString(map.keySet().toArray(new int[0]));
}
```

http://127.0.0.1/user/valuOfMap?map:1=paoding&map:2=rose

```java
public String valueOfMap(@Param("map") Map<Integer, String> map) {
	return "@" + Arrays.toString(map.values().toArray(new String[0]));
}
```

http://127.0.0.1/user/map?map:1=paoding&map:2=rose 

```java
public String map(@Param("map") Map<Integer, String> map) {
	StringBuilder sb = new StringBuilder();
	for (Map.Entry<Integer, String> entry : map.entrySet()) {
		sb.append(entry.getKey()).append("=").append(entry.getValue()).append("<br>");
	}
	return "@" + sb;
}
```

### 表单提交
http://127.0.0.1/user?id=1&name=rose

```java
@Post
public String post(User user) {
	return "@" + user.getId() + "=" + user.getName();
}
```

### 设置内嵌对象的属性值
POST http://127.0.0.1/user?id=1&name=rose&level.id=3

```java
@Post
public String post(User user) {
	return "@" + user.getId() + "; level.id=" + user.getLevel().getId();
}
```

### 返回JSON
http://127.0.0.1/user/json?id=1

http://127.0.0.1/user/json?id=2

```java
public class UserController {
    public Object json(@Param("id") String id) {

        JSONObject json = new JSONObject();

        json.put("id", id);

        json.put("name", "rose");

        json.put("text", "可以有中文");

        // rose将调用json.toString()渲染

        return json;
    }

    // 把JSONObject放到方法中，Rose将帮忙创建实例
    public Object json2(JSONObject json, @Param("id") String id) {

        json.put("id", id);

        json.put("name", "rose");

        json.put("text", "可以有中文");

        // rose将调用json.toString()渲染

        return json;
    }
}
```

### 返回XML
http://127.0.0.1/user/xml

```java
public class UserController {
	public Object xml(Invocation inv) {

        User user = new User();

        user.setId(1);

        user.setName("rose");

        inv.addModel("user", user);

        // rose将调用user-xml.xml或.vm或.jsp渲染页面(按字母升序顺序优先: jsp, vm, xml)

        // 使用user-xml.xml的，默认contentType是text/xml;charset=UTF-8，语法同velocity

        // 使用user-xml.jsp或user-xml.vm的可在本方法中标注@HttpFeatures(contentType="xxx")改变

        // jsp的也可通过<%@ page contentType="text/html;charset=UTF-8" %>改变

        return "user-xml";
    }
}
```

**views/user-xml.xml**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<user>
    <id>$user.id</id>
    <name>$user.name</name>
</user>
```

如果是使用DOM构建xml的，则建议在所在的controllers下创建DocumentInterceptor拦截器，
控制器直接返回Document对象，

http://127.0.0.1/user/xml2

```java
package com.xiaonei.rose.usage.controllers;

public class DocumentInterceptor extends ControllerInterceptorAdapter {
    @Override
    public Object after(Invocation inv, Object instruction) throws Exception {
        if (instruction instanceof Documenet) {
            Docuement doc = (Docuement) instruction;
            HttpServletResponse response = inv.getResponse();
            if (response.getContentType() == null) {
                response.setContentType("text/xml;charset=UTF-8");
            }
            document.write(response.getWriter());
            return ""; // 返回空串给Rose，让其不用再管render的事情了
        }
        return instruction; 
    }
}
```

```java
import org.dom4j.Document;
import org.dom4j.DocumentHelper;
import org.dom4j.DomDocument;
import org.dom4j.Element;

public class UserController {
    public Object xml2(Invocation inv) {
        Document doc = new DomDocument();
        Element listElement = doc.addElement("user-list");
        Element user1Element = listElement.addElement("user");
        user1Element.addAttribute("id", "1");
        user1Element.addAttribute("name", "paoding");
        Element user2Element = listElement.addElement("user");
        user1Element.addAttribute("id", "2");
        user1Element.addAttribute("name", "rose");
        return doc;
    }
}
```

### 参考
* [Rose_Code_Fragment_Controller](https://code.google.com/archive/p/paoding-rose/wikis/Rose_Code_Fragment_Controller.wiki)
* [Rose_Code_Fragment_Controller2](https://code.google.com/archive/p/paoding-rose/wikis/Rose_Code_Fragment_Controller2.wiki)
[Rose_Code_Fragment_Controller3](https://code.google.com/archive/p/paoding-rose/wikis/Rose_Code_Fragment_Controller3.wiki)