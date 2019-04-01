# Java 深入编程

## Java的数组

### 声明数组

```java
String[] name;// 首选的方法
String name[];// 效果相同，但不是首选方法
```

数组的元素是通过索引访问的。数组索引从 0 开始，所以索引值从 0 到 length-1

遍历数组

```java
for(String e: name)
{
    System.out.println(e);
}
```

### 多维数组

```java
int a[][] = new int[2][3];
```

## 异常处理

Checked Expection：编译时异常，需要处理

Runntime Expection：运行时异常

### 自定义异常

需要继承Excption

### throws/throw关键词

```java
// 类 或者 方法
throws IOException
// 方法内部
throw new IOException();
```



## 常用的类

### Statement类

Statement 对象用于将 SQL 语句发送到数据库中。实际上有三种 Statement 对象，它们都作为在给定连接上执行 SQL 语句的包容器

1. Statement：发送简单的不带参数的、或者拼接的SQL，存在SQL注入风险
2. PreparedStatement：预编译，防止SQL注入
3. CallableStatement：存储过程

