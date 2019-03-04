# JavaWeb开发

## 安装Tomcat(Windows)

Tomcat 是轻量级应用服务器，开源、稳定、资源占用小。

| 目录         | 说明                                                         |
| ------------ | ------------------------------------------------------------ |
| **/bin**     | 存放各种平台下用于启动和停止Tomcat的脚本文件                 |
| **/conf**    | 存放**Tomcat**服务器的各种配置文件                           |
| **/lib**     | 存放**Tomcat**服务器所需的各种**JAR**文件                    |
| **/logs**    | 存放**Tomcat**的日志文件                                     |
| **/temp**    | **Tomcat**运行时用于存放临时文件                             |
| **/webapps** | 当发布**Web**应用时，默认情况下会将**Web**应用的文件   存放于此目录中 |
| **/work**    | **Tomcat**把由**JSP**生成的**Servlet**放于此目录下           |

下载安装包，不再说明了，本例子使用的是`apache-tomcat-8.5.13`。

修改`conf\server.xml`修改tomcat的监听地址。

```xml
 <Connector port="9090" protocol="HTTP/1.1"
               connectionTimeout="20000"
               redirectPort="8443" />
```

修改`bin/setclasspath.bat`添加jdk路径

```bash
rem ---------------------------------------------------------------------------
rem Set JAVA_HOME or JRE_HOME if not already set and ensure any provided
rem settings are valid and consistent with the selected start-up options.
rem ---------------------------------------------------------------------------
set JAVA_HOME=D:\Application\Program Files\Java\jdk1.8.0_191
set JRE_HOME=D:\Application\Program Files\Java\jre1.8.0_191
```

cmd中运行`service.bat install`安装windows服务，`service.bat uninstall`卸载服务。

```bash
# 启动
bin\startup.bat
# 关闭
bin\shutdown.bat
```

在IDEA中配置tomcat，需要先启用tomcat插件，配置如图，可以热部署，同时需要修改Deployment项中Application Context的名称。

![1551698702448](assets\1551698702448.png)

部署war包直接放在`webapps`目录下，重启服务即可，注意：运行状态不可删除*.war。

## JSP

运行在服务器端的Java页面，使用HTML嵌套Java代码实现，即为 Java Server Pages。

JSP中默认使用的字符编码方式：iso-8859-1

## JSP处理过程

浏览器发送一个 HTTP 请求给服务器，Web 服务器识别出这是一个对 JSP 网页的请求，并且将该请求传递给 JSP 引擎，通过使用 URL或者 .jsp 文件来完成。JSP 引擎从磁盘中载入 JSP 文件，然后将它们转化为 Servlet。这种转化只是简单地将所有模板文本改用 println() 语句，并且将所有的 JSP 元素转化成 Java 代码。JSP 引擎将 Servlet 编译成可执行类，并且将原始请求传递给 Servlet 引擎。Web 服务器的某组件将会调用 Servlet 引擎，然后载入并执行 Servlet 类。在执行过程中，Servlet 产生 HTML 格式的输出并将其内嵌于 HTTP response 中上交给 Web 服务器。Web 服务器以静态 HTML 网页的形式将 HTTP response 返回到浏览器中。最终，Web 浏览器处理 HTTP response 中动态产生的HTML网页，就好像在处理静态网页一样。

可以形象说明：请求-->index.jsp=>index_jsp.java=>index_jsp.class

### JSP生命周期

以下是JSP生命周期中所走过的几个阶段：

- **编译阶段**：servlet容器编译servlet源文件，生成servlet类
- **初始化阶段**：加载与JSP对应的servlet类，创建其实例，并调用它的初始化方法
- **执行阶段**：调用与JSP对应的servlet实例的服务方法
- **销毁阶段**：调用与JSP对应的servlet实例的销毁方法，然后销毁servlet实例

> JSP编译

当浏览器请求JSP页面时，JSP引擎会首先去检查是否需要编译这个文件。如果这个文件没有被编译过，或者在上次编译后被更改过，则编译这个JSP文件。

> JSP初始化

容器载入JSP文件后，它会在为请求提供任何服务前调用jspInit()方法。如果您需要执行自定义的JSP初始化任务，复写jspInit()方法就行了。

## JSP语法

```jsp
<%= (new java.util.Date()).toLocaleString()%>
<%out.print("输出语句");%>
<%out.println("输出语句");>

<%-- 执行如下代码，反复刷新JSP页。 --%> 
<%! int x=10;%><%-- 全局变量的写法。 --%> 
<%
  int y=20;
  out.print("x="+x++);
  out.print("y="+y++);
%>
```

> 中文编码

```jsp
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
```

> JSP表达式

```jsp
<%= (new java.util.Date()).toLocaleString()%>
```

> JSP注释

```jsp
<%-- 该部分注释在网页中不会被显示 --%> 
```

> JSP指令

| **指令**           | **描述**                                                  |
| ------------------ | --------------------------------------------------------- |
| <%@ page ... %>    | 定义页面的依赖属性，比如脚本语言、error页面、缓存需求等等 |
| <%@ include ... %> | 包含其他文件                                              |
| <%@ taglib ... %>  | 引入标签库的定义，可以是自定义标签                        |

> JSP隐藏对象

| **对象**    | **描述**                                                     |
| ----------- | ------------------------------------------------------------ |
| request     | **HttpServletRequest**类的实例                               |
| response    | **HttpServletResponse**类的实例                              |
| out         | **PrintWriter**类的实例，用于把结果输出至网页上              |
| session     | **HttpSession**类的实例                                      |
| application | **ServletContext**类的实例，与应用上下文有关                 |
| config      | **ServletConfig**类的实例                                    |
| pageContext | **PageContext**类的实例，提供对JSP页面所有对象以及命名空间的访问 |
| page        | 类似于Java类中的this关键字                                   |
| Exception   | **Exception**类的对象，代表发生错误的JSP页面中对应的异常对象 |

## HttpServletRequest

request对象是javax.servlet.http.HttpServletRequest类的实例。每当客户端请求一个页面时，JSP引擎就会产生一个新的对象来代表这个请求。

## 过滤器

## 监听器

## 文件上传和下载

