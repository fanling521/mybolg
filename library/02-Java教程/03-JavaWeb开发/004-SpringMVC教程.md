# SpringMVC教程

## 什么是SpringMVC

## SpringMVC的原理

## SpringMVC简单用法

#### 前端控制器

新增`web.xml`

```xml
<servlet>
    <!--配置前端控制器-->
	<servlet-name>dispatcherServlet</servlet-name>
	<servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
    <!--指定配置文件的路径-->
	<init-param>
		<param-name>contextConfigLocation</param-name>
		<param-value>classpath:springmvc.xml</param-value>
	</init-param>
    <!--启动优先级-->
	<load-on-startup>2</load-on-startup>
</servlet>
<servlet-mapping>
	<servlet-name>dispatcherServlet</servlet-name>
    <!--拦截所有请求-->
	<url-pattern>/</url-pattern>
</servlet-mapping>
```

#### 视图解析器

新增`springmvc.xml`

```xml
<!--扫描包路径-->
<context:component-scan base-package="com.fanling.springmvc01"/>
<!--视图解析器-->
<bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
	<property name="prefix" value="/view/"></property>
	<property name="suffix" value=".jsp"></property>
</bean>
```

#### 映射请求

##### @RequestMapping

Spring MVC 使用 @RequestMapping 注解为控制器指定可以处理哪些 URL 请求。

在控制器的类定义及方法定义处都可标注 @RequestMapping。

DispatcherServlet 截获请求后，就通过控制器上 @RequestMapping 提供的映射信息确定请求所对应的处理方法。

- value：请求的url
- method：请求方法
- params：请求参数

##### @RequestParam

使用 @RequestParam 绑定请求参数。

```java
@RequestMapping("/show3")
public String show3(@RequestParam String name){
	System.out.println("name="+name);
	return "ok";
}
```

##### @PathVariable

映射 URL 绑定的占位符。

```java
@RequestMapping("/show/{id}")
public String show4(@PathVariable String id){
    System.out.println("id="+id);
	return "ok";
}
```

##### 处理模型

- Model
- ModelAndView
- Map

#### 数据校验

新增引用

```xml
<dependency>
	<groupId>org.hibernate.validator</groupId>
	<artifactId>hibernate-validator</artifactId>
	<version>6.0.13.Final</version>
</dependency>
```

添加全局配置文件

```xml
<mvc:annotation-driven/>
```

实体类中的使用

```java
@NotEmpty(message = "用户名不能为空")
private String userName;
@NotEmpty(message = "密码不能为空")
@Pattern(regexp = "/^\\d{5,8}$/",message = "密码格式不匹配")
private String password;
@Email(message = "邮箱不能为空")
private String email;
```

controller的使用

```java
public String show5(@Validated User user, BindingResult br, Model model){
if(br.hasErrors()){
	br.getFieldErrors().forEach(x->
                                model.addAttribute(x.getField(),x.getDefaultMessage()));
	}else{
		model.addAttribute("result","ok");
	}
	return "show5";
}
```

#### 处理静态资源

SpringMVC会默认将静态资源的url拦截。

**解决方法：**

将所有的静态资源文件放在一个文件夹下。如`static`

```xml
<!--处理静态资源-->
<mvc:resources mapping="/static/**" location="/static/"/>
```

## SpringMVC应用

### 处理JSON

添加依赖引用

```xml
<dependency>
	<groupId>com.fasterxml.jackson.core</groupId>
	<artifactId>jackson-databind</artifactId>
	<version>2.9.7</version>
</dependency>
```

controller层的新增

```java
	@RequestMapping("/json")
    @ResponseBody
    public User json(){
        User user=new User();
        user.setId(1);
        user.setUserName("fanling");
        user.setPassword("123456");
        user.setEmail("998@qq.com");
        return user;
    }
```

返回的信息

```json
{"id":1,"userName":"fanling","password":"123456","email":"998@qq.com"}
```



