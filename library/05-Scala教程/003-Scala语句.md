# Scala 语句

## Scala控制台输入

使用`StdIn`

```scala
def main(args: Array[String]): Unit = {
    println("请输入您的姓名：")
    val name:String = StdIn.readLine()
    println("请输入您的年龄：")
    var age:Int=StdIn.readInt()
    println("请输入您的薪水：")
    var sal:Float=StdIn.readFloat()
    printf("年龄%s,年龄：%d，薪水：%.2f",name,age,sal)
  }
```

## 流程控制

### 顺序控制

和Java一样

### 分支控制

#### 单分支

```scala
def main(args: Array[String]): Unit = {
    printf("输入你的年龄：")
    val age:Int=StdIn.readInt()
    if(age>18){
      printf("成年人了")
    }
  }
```

#### 双分支

```scala
def main(args: Array[String]): Unit = {
    printf("输入你的年龄：")
    val age:Int=StdIn.readInt()
    if(age>18){
      printf("成年人了")
    }else{
      printf("还是小孩")
    }
  }
```

### 多分支

和Java一样

#### 特别的注意点

Scala中任意表达式都是有返回值的，也就意味if-else有返回值，取决于代码块中的最后一行。

```scala
// 输入20 输出40
def main(args: Array[String]): Unit = {
  var age:Int=StdIn.readInt()
  val res=if(age>18){
    "hello"
    28+12
  }else{
    "hh"
  }
  printf("res="+res)
}
// 输入 11 ，输出 ()
def main(args: Array[String]): Unit = {
  var age:Int=StdIn.readInt()
  val res=if(age>18){
    "hello"
    28+12
  }
  printf("res="+res)
}
```

Scala中没有switch！

### 循环控制

#### for的推导式

**第一种**：范围->[1,3]

```scala
  def main(args: Array[String]): Unit = {
    for(i<-1 to 3){
      System.out.println("第"+i+"个");
    }
  }
```

**第二种**：范围->[1,3)

```scala
for(i<-1 until 3){
	System.out.println("第"+i+"个");
}
```

**循环守卫**：满足条件就进入循环体内，不满足就跳过。

```scala
for(i<-1 to 3 if i==2){
	System.out.println("第"+i+"个");
}
```

**引入变量**：需要添加`;`来隔断逻辑

```scala
for(i<-1 to 3;j=i-1){
	System.out.println("j="+j);
}
println("===========等价于===========")
for(i<-1 to 3){
    for(j<-1 to 3){
      	System.out.println("i="+i+",j="+j);  
    }
}
```

**循环嵌套**：除了传统的方式，Scala支持以下语法

```scala
for(i<-1 to 3;j <-1 to 3){
	printf("i="+i+",j="+j+"\n")
}
```

**循环返回值**：Vetor集合

```scala
def main(args: Array[String]): Unit = {
   val res= for(i<- 1 to 10) yield {
      if(i%2==0){
        i+"-偶数"
      }else{
        i+"-奇数"
      }
    }
    println(res)
  }
```

返回：`Vector`(1-奇数, 2-偶数, 3-奇数, 4-偶数, 5-奇数, 6-偶数, 7-奇数, 8-偶数, 9-奇数, 10-偶数)

**控制步长**：两种方式来完成控制步长`2`

```scala
def main(args: Array[String]): Unit = {
    println("用Range(开始，结束，步长)-------")
    for(i<-Range(1,10,2)){
      println(i)
    }
    println("---用循环守卫--------")
    for(i<- 1 to 10 if i%2==1){
      println(i)
    }
  }
```



**注意点**：

1. 小括号`()`可以替换为`{}`

#### While循环

基本语法和Java一样，在Scala中没有返回值，由于内部需要对外部变量进行影响，不推荐使用`while`循环。

#### Do-While循环

和While的描述一样。

