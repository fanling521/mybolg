# Lombok教程

## 什么是Lombok

Lombok想要解决了的是在我们实体Bean中大量的Getter/Setter方法，以及toString, hashCode等可能不会用到，但是某些时候仍然需要复写，以期方便使用的方法；在使用Lombok之后，将由其来自动帮你实现代码生成，注意，其是 **在运行过程中，帮你自动生成的** 。就是说，将极大减少你的代码总量。

- 消除模板代码
- 便捷的生成比较复杂的代码

### SpringBoot中使用Lombok

（1）添加依赖

```xml
<dependency>
  <groupId>org.projectlombok</groupId>
  <artifactId>lombok</artifactId>
  <version>1.18.8</version>
   <scope>provided</scope>
</dependency>
```

（2）测试类

```java
@Data
public class SysConfig {
    private String name;
    private String version;
}
```

## 常用的注解

- @NonNull : 让你不在担忧并且爱上NullPointerException

- @CleanUp : 自动资源管理：不用再在finally中添加资源的close方法

- @Setter/@Getter : 自动生成set和get方法

- @ToString : 自动生成toString方法

- @EqualsAndHashcode : 从对象的字段中生成hashCode和equals的实现

- @NoArgsConstructor/@RequiredArgsConstructor/@AllArgsConstructor
  自动生成构造方法

- @Data : 自动生成set/get方法，toString方法，equals方法，hashCode方法，不带参数的构造方法

- @Value : 用于注解final类

- @Builder : 产生复杂的构建器api类

- @SneakyThrows : 异常处理（谨慎使用）

- @Synchronized : 同步方法安全的转化

- @Getter(lazy=true) :

- @Log : 支持各种logger对象，使用时用对应的注解，如：@Log4j  

### 使用技巧

自动生成方法的变量。举例： 比如在这里，我们只想让stuName,stuAge两个成员变量出现在 toString 方法里。

```java
@ToString(of = {"stuName","stuAge"})
```

为了使代码更加简洁， Lombok永续我们在类级上使用这些注解。如果这些注解放在类名之上，

```java
@Setter
@Getter
public class Student {}
```

**@Buidler** :实例化这个类

```java
@Builder
public class Student {}

Student student = Student.builder()
	.stuName("张三");
```

