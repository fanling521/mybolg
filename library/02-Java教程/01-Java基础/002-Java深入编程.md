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

