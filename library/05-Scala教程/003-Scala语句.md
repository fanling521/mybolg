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

