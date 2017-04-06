# 自定义http参数支持

* 我们把一个controller的类里的一个方法叫做action，它实际对应用户看到的一个url。
* 在action里可以接收各种各样的参数，也可以自己定义需要的参数。
* rose自己定义了一些常见的类型，基本上很有机会会用到自己定义，但是在某些情况下，也是个不错的选择：
* 用来对指定的参数类型的值进行固定的修改和赋值。

#### 示例

MiBeanResolver.java放在controllers目录下：

```java
public class ChenBeanResolver implements ParamResolver {
    @Override
    public Object resolve(Invocation inv, ParamMetaData metaData) throws Exception {
        for (String paramName : metaData.getParamNames()) {
            if (paramName != null) {
                Mi mi = new Mi();
                String value1 = inv.getParameter("mi1");
                String value2 = inv.getParameter("mi2");
                mi.setMi1(value1);
                mi.setMi2(value2);
                return mi;
            }
        }
        return null;
    }  
    @Override
    public boolean supports(ParamMetaData metaData) {
        return Mi.class == metaData.getParamType();
    }
}
```

上述代码的意思：

* 如果在action里一个参数的类型是Mi(com.xiaomi.model.Mi)，就会走这个resolver，这里对两个参数进行了组装。
* 用户如果访问的参数里传入了mi1和mi2的值，则会直接组装出来一个Mi对象。

配合上述resolver的action代码为：
```java
@Get("/param")  
   public String param(Mi mi) throws Exception {  
       return "@hello world:" + mi.getMi1() + ":" + mi.getMi2();  
   }  
```
用户访问：http://127.0.0.1/param?chen1=1&chen2=2 将会返回：`hello world:1:2`

#### Rose内置的参数支持
除了上述的自定义resolver外，rose还内置了丰富的resolver，都是大家的经验总结，使用起来会非常方便，它们是：
* 所有的基础java类型，都可以直接使用，rose进行自动转换，比如在action中的类型为long id，则id可以转为数字，不再需要从string转为long。
* array/map/bean同样可用，它们的接收参数规则为：
	* ?id=1,2,3,4 或者 ?id=1&id=2&id=3 对应 @Param("id") int[] idArray
	* ?map:1=paoding&map:2=rose 对应 @Param("map") Map map
* POST http://127.0.0.1/user?id=1&name=rose&level.id=3 对应接收代码：

```java
@Post  
public String post(User user) {  
    return "@" + user.getId() + "; level.id=" + user.getLevel().getId();  
}  
```
代码中User是一个自定义的bean，有属性id,name,level等。

### 参考
* [自定义http参数支持](http://www.54chen.com/java-ee/rose-3-4.html)