# Spring教程

## 什么是Spring

Spring是一个开源框架，是为了解决企业应用开发的复杂性而创建的，为了解决企业应用开发的复杂性而创建的，是一个分层的`JavaEE`一站式轻量级开源框架。

### Spring优点

- 方便解耦，简化开发：通过Spring提供的IoC容器，我们可以将对象之间的依赖关系交由Spring进行控制
- AOP编程的支持：通过Spring提供的AOP功能，方便进行面向切面的编程
- 声明事物的支持：只需要通过配置就可以完成对事务的管理
- 方便集成各种优秀框架

### Spring的使用

```xml
<dependencies>
        <!--spring-beans-->
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-beans</artifactId>
            <version>${spring.version}</version>
        </dependency>
    	<!--spring-core-->
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-core</artifactId>
            <version>${spring.version}</version>
        </dependency>
    	<!--spring-context-->
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-context</artifactId>
            <version>${spring.version}</version>
        </dependency>
    	<!--spring-expression-->
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-expression</artifactId>
            <version>${spring.version}</version>
        </dependency>
</dependencies>
```

## Spring核心

### Spring的控制反转和依赖注入

#### 什么是控制反转



#### 什么是依赖注入



#### 基于XML的注入方式

##### set方式注入

```xml
<bean id="stu" class="com.fanling.entity.Student">
        <property name="id" value="1020"/>
        <property name="name" value="樊领"/>
        <property name="user" ref="user"/>
        <property name="list">
            <list>
                <value>惊奇队长1</value>
                <value>惊奇队长2</value>
                <value>惊奇队长3</value>
            </list>
        </property>
        <property name="map">
            <map>
                <entry key="a" value="A"></entry>
                <entry key="b" value="B"></entry>
                <entry key="c" value="C"></entry>
            </map>
        </property>
</bean>
```

##### 构造函数

```xml
<bean id="user2" class="com.fanling.entity.User">
	<constructor-arg value="12"/>
	<constructor-arg value="name2"/>
</bean>
```

##### 静态工厂

```java
public class StudentFactory {
    public static Student getStudent(){
        return new Student();
    }
}
```

```xml
<!--静态工厂-->
    <bean id="stu1" class="com.fanling.entity.StudentFactory" factory-method="getStudent"></bean>
```

##### 实例工厂

#### 作用域

作用域：用于确定spring创建bean实例个数

- singleton 单例
- prototype 多例

#### 生命周期

- 实例化bean对象(通过构造方法或者工厂方法)
- 设置对象属性(setter等)（依赖注入）
- 检查Aware接口
- 将Bean实例传递给Bean的前置处理器的postProcessBeforeInitialization方法
- 调用Bean的初始化方法
- 将Bean实例传递给Bean的后置处理器的postProcessAfterInitialization方法
- 使用Bean
- 容器关闭之前，调用Bean的销毁方法
```xml
<!--允许自定义初始化和销毁方法-->
<bean  id="" class="" init-method="初始化方法名称"  destroy-method="销毁的方法名称">
```

#### 基于注解的装配Bean

在配置文件中使用

```xml
<context:component-scan base-package="com.fanling.bean"></context:component-scan>
```

##### Component

@Component 取代了`<bean class=""></bean>`

@Component(“id”) 取代了`<bean id="id" class=""></bean>`

##### 3个衍生注解

- @Repository ：dao层
- @Service：service层
- @Controller：web层

##### 常用的属性注入

- 按类型 @Autowired
- 按名称 @Qualifier("名称")
- 按名称 @Resource(name="名称")