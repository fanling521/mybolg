# Spring教程

## 什么是Spring

Spring是一个开源框架，是为了解决企业应用开发的复杂性而创建的，为了解决企业应用开发的复杂性而创建的，是一个分层的`JavaEE`一站式轻量级开源框架。

### Spring优点

- 方便解耦，简化开发：通过Spring提供的IoC容器，我们可以将对象之间的依赖关系交由Spring进行控制
- AOP编程的支持：通过Spring提供的AOP功能，方便进行面向切面的编程
- 声明事物的支持：只需要通过配置就可以完成对事务的管理
- 方便集成各种优秀框架

### 简化Java开发

- 基于POJO轻量级和最小侵入式开发
- 通过依赖注入和面向接口实现松耦合
- 基于切面和惯例进行声明式编程
- 通过切面和模板**减少样板式代码 **

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

Spring的核心是控制反转和AOP。

### Spring的控制反转

#### 什么是控制反转（IoC）

简单说，就是交给spring的IoC容器管理对象，各个对象之间没有关联，

#### 什么是依赖注入（DI）

简单说，由IoC容器在运行期间，动态地将某种依赖关系注入到对象之中，实现对象之间的解耦。

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

#### Bean的作用域

作用域`scope`：用于确定spring创建bean实例个数

- singleton 单例
- prototype 多例

#### Bean的生命周期

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

##### Component注解

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

### Spring的AOP编程

功能： 让关注点代码与业务代码分离。

面向切面编程就是指： **对很多功能都有的重复的代码抽取，再在运行的时候往业务方法上动态植入“切面类代码”。**

#### JDK动态代理

代理类与委托类实现同一接口，主要是通过代理类实现InvocationHandler并重写invoke方法来进行动态代理的，在invoke方法中将对方法进行增强处理

####  CGLIB动态代理机制

对代理对象类的class文件加载进来，通过修改其字节码生成子类来处理。

CGLIB代理也叫子类代理，从内存中构建出一个子类来扩展目标对象的功能。

#### AspectJ编程

##### 切点表达式

execution(修饰符  包名.类名.方法名(参数) throws 异常)

返回值，可用`*`模糊的类方法路径，`(..)`表示方法不定的参数，throws一般不写。

##### 通知类型

- before:前置通知
- afterReturning:后置通知
- around:环绕通知
- afterThrowing:抛出异常通知
- after:最终通知

##### 基于XML的配置

重点：**proxy-target-class="true"**属性

```xml
<!--目标对象-->
<bean id="userDaoImpl" class="com.fanling.xmlaop.dao.impl.UserDaoImpl"></bean>
<!--切面对象-->
<bean id="myadvice" class="com.fanling.xmlaop.UserAdvice"></bean>
<!--aop配置-->
<aop:config proxy-target-class="true">
		<aop:aspect ref="myadvice">
	<!--定义切点-->
		<aop:pointcut id="pointcut01" expression="execution(* com.fanling.xmlaop.*Impl.updateUser(..))"/>
	<!--通知类型-->
		<aop:before method="beforeAdvice" pointcut-ref="pointcut01"/>
		<aop:after-returning method="afterReturning" pointcut-ref="pointcut01" returning="object"/>
		<aop:after-throwing method="afterThrowing" pointcut-ref="pointcut01" throwing="throwable"/>
		<aop:around method="around" pointcut="execution(* com.fanling.xmlaop.dao.impl.UserDaoImpl.addUser(..))"/>
		<aop:after method="afterAdvice" pointcut-ref="pointcut01"/>
	</aop:aspect>
</aop:config>
```

##### 基于注解的配置

重点：**proxy-target-class="true"**的属性

```xml
<!--全局扫描-->
<context:component-scan base-package="com.fanling.aop"/>
<!--自动代理AOP-->
<aop:aspectj-autoproxy proxy-target-class="true"/>
```

```java
@Component
@Aspect
public class BookAdvice {

    @Pointcut(value = "execution(* com.fanling.aop.dao.impl.*Impl.*(..))")
    public void myPoint(){}

    @Before(value = "myPoint()")
    public void before(JoinPoint joinPoint) {
        System.out.println("前置通知");
    }

    @AfterReturning(value = "myPoint()",returning ="o" )
    public void afterReturning(JoinPoint joinPoint,Object o) {
        System.out.println("后置通知");
    }

    @Around(value = "execution(* com.fanling.aop.dao.impl.*Impl.addBook(..))")
    public Object around(ProceedingJoinPoint pjt) {
        Object o = null;
        System.out.println("开始环绕通知");
        try {
            o = pjt.proceed();
        } catch (Throwable throwable) {
            throwable.printStackTrace();
        }
        System.out.println("结束环绕通知");
        return o;
    }

    @AfterThrowing(value = "execution(* com.fanling.aop.dao.impl.*Impl.updateBook(..))",throwing = "throwable")
    public void afterThrowing(JoinPoint joinPoint, Throwable throwable) {
        System.out.println("异常通知");
    }

    @After(value = "myPoint()")
    public void after() {
        System.out.println("最后通知");
    }
}
```

注意事项：

- 若目标函数有返回值，又使用了`@around`通知，需要添加`Object`返回值
- Spring5的AOP使用需要在全局配置文件中添加**proxy-target-class="true"**属性