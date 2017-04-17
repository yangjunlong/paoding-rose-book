# 接入fis-velocity-tools工具包

	fis-velocity-tools 是百度前端技术团队开发的针对服务端为 JAVA + Velocity 的前端集成解决方案后端工具包。

#### Rose + fis-velocity-tools 整合

`webapp/WEB-INF` 目录下新建：`fis.properties`、`velocity.properties`

* 新建 fis.properties

默认 map json 文件是从 /WEB-INF/config 文件夹下读取的，如果有修改存放地址，则需要添加一个 fis.properties 文件到 /WEB-INF/ 目录。 内容如下：

```
mapDir = /WEB-INF/config
```

* 新建 velocity.properties

```
userdirective = com.baidu.fis.velocity.directive.Html, com.baidu.fis.velocity.directive.Head, com.baidu.fis.velocity.directive.Body, com.baidu.fis.velocity.directive.Require, com.baidu.fis.velocity.directive.Script, com.baidu.fis.velocity.directive.Style, com.baidu.fis.velocity.directive.Uri, com.baidu.fis.velocity.directive.Widget, com.baidu.fis.velocity.directive.Block, com.baidu.fis.velocity.directive.Extends, com.baidu.fis.velocity.directive.Parent, com.baidu.fis.velocity.directive.Filter
```

* 在web.xml文件里添加如下：

```
<listener>
    <listener-class>com.baidu.fis.servlet.MapListener</listener-class>
</listener>
```

* 在applicationContext.xml文件里添加如下：

为了让 fis 自定义的 directive 能够正常读取 map.json 文件，需要添加一个 bean 初始化一下。

```
<!--初始 fis 配置-->
<bean id="fisInit" class="com.baidu.fis.velocity.spring.FisBean" />
```

### 参考
[jello](https://github.com/fex-team/jello)