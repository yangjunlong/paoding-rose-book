# 统一的参数验证办法

	我们把的参数验证办法叫ParamValidator
	一般来说，像比如说验证http传来的参数是不是为空呀啥的（发挥你的想象力）。
	好处在于不用再重复地写if else

#### 举个栗子

来看一个例子，验证用户的参数不可为空(灰常灰常的实用)：

```java
public class NotBlankParamValidator implements ParamValidator {
    @Override
    public boolean supports(ParamMetaData metaData) {
        return metaData.getAnnotation(NotBlank.class) != null;
    }
    @Override
    public Object validate(ParamMetaData metaData, Invocation inv, Object target, Errors errors) {
        String paramName = metaData.getParamName();
        String value = inv.getParameter(paramName);
        if (StringUtils.isBlank(value)) {
            return "@参数不能为空";
        }
        return null;
    }
}
```

* 放到controllers下
* 实现ParamValidator
* 实现supports方法，这个方法用来做判断是否要验证当前得到的http参数，一般都用个注解来判断比较文艺
* 实现validate方法，这里是主要逻辑
* metaData里放的是参数的原型
* inv是rose的基础调用
* target是这个参数的最后解析结果，参看上一节里提到的东西
* errors是这个参数解析时出来的错误
* NotBlank是一个自己定义的annotation

#### 使用时action长什么样？

下面的代码是action中使用时长的样子:

```java
@Get("/notBlank")  
public String notBlank(@NotBlank @Param("messages") String messages) throws Exception { 
    return "@hello world";
}
```

* 当遇到NotBlank注解的参数时，会自动执行参数判断
* 如果messages为空，则会得到“参数不能为空”的返回

### 参考
* [controller层：统一的参数验证办法](http://www.54chen.com/java-ee/rose-3-5.html)