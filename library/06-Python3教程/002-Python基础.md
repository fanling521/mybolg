# Python基础

## Python函数

**语法**：

```
def [函数名]():
	"""函数注释"""
	[函数执行代码]
```

**注意点**：

（1）函数之间需要保留2个空行

（2）函数的注释写在函数体中，用文档注释

（3）实参，形参，返回值和Java的概念是一样，不再说明

```python
def sum(num1, num2):
    """两个数字，求和"""
    return num1 + num2


sum_result = sum(10, 22)
print(sum_result)
```

## Python模块

- 以`.py`结尾的文件都是模块，模块也是标识符，不可以数字命令开头
- `.pyc` 是编译过的引用模块
- 引用模块，`import xx`

## 高级变量类型

### 列表list

语法：

```python
list=["xx","xx2"]
# 获取元素，用下标，和Java数组类似
```

**取值**：根据索引值取值即可

**确定元素位置**：list.index(元素)

**增加**：

（1）`list.append(元素)`–->尾部新增元素

（2）`list.insert(index,元素)`–->指定索引位置新增元素

（3）`list.extend(list)`–>追加新的列表

**删除**：

（1）`list.pop([index])`–->默认删除最后一个元素，可以指定索引位置

（2）`list.remove(元素)`–->根据元素删除

（3）使用关键字 `del 变量名`

（4）`list.clear()` 清空列表

**统计：**

（1）`len(list)`：统计元素个数

（2）某个字符出现的字数：`list.count(元素)`

**排序：**

（1）升序排序：`list.sort()`

（2）降序排序：`list.sort(reverse=True)`

（3）反转：`list.reverse()`

**迭代遍历：**

```python
list = ["苹果", "西瓜", "香蕉", "哈密瓜"]
for name in list:
    print(name)
    
# 完整的for循环语法
for xx in xx:
    # 循环
else:
    # 循环完全结束的执行
```

### 元组tuple

（1）定义，和列表类似，元素不可修改

```python
tuple=("12",2,"22")
# 只有一个元素
tuple=("12",)
# 取值 和 操作
into_tuple = (23, "32",23)
print(into_tuple[1])
print(into_tuple.count(23))
print(into_tuple.index("32"))
```

（2）元组和列表相互转化

```python
num_list = [1, 2, 3, 4]
num_tuple = tuple(num_list)
print(type(num_tuple))
num_list2 = list(num_tuple)
print(type(num_list2))
```

### 字典dictionary

（1）定义

使用键值对，用`{}`包裹

```python
xiaoming = {"name": "小明", "age": "12"}
# 取值
print(xiaoming["name"])
# 删除
xiaoming.pop("age")
# 更新
xiaoming2 = {"height": "134"}
xiaoming.update(xiaoming2)
print(xiaoming)
```

（2）遍历字典

```python
xiaoming = {"name": "小明", "age": "12"}
for k in xiaoming:
    print("%s-%s" % (k, xiaoming[k]))
```

### 字符串

内置了大量的函数，需要的时候可以查看API，使用方法和前面的几种类似。

Python 支持转义符。

```
str.isdecimal -- 只针对数字
str.isdigint---特殊符号+数字，全角数字和unicode字符
str.isnumeric--中文，全角，数字
```

（1）字符串的格式化

```python
str.ljust(10,"") # 左对齐
str.rjust(10,"") # 右对齐
str.center(10,"") # 居中
# ===============================
list=["静夜思",
      "李白",
      "窗前明月光",
      "疑是地上霜",
      "举头望明月",
      "低头思故乡"]
for str in list:
    print("|%s|" % str.center(10,chr(12288)))
    
for str in list:
    print("|%s|" % str.ljust(10,chr(12288)))

for str in list:
    print("|%s|" % str.rjust(10,chr(12288)))
```

（2）去除空白字符

```python
str.strip() # 去掉两边
str.lstrip()# 去掉左边
str.rstrip()# 去掉右边
# ===============================
str = " hello "
print(str.rstrip())
print(str.lstrip())
print(str.strip())
str1="hello"
print(str1.rstrip('o'))
```

（3）字符串拆分和拼接

```python
str.partition("str") #将字符串分成3个元素的元组，str前面、str、str后面
str.rpartition("str") # 从右边拆分
# ===================================
string="helloworld"
tuple_str=string.partition("ll")
print(tuple_str)
# ('he', 'll', 'oworld')
# ==============================
string2="hello world hello python"
print(string2.split(" ",1))
# split 带数字参数，表示分几组，
# ['hello', 'world hello python']
print(string2.split(" "))
# ['hello', 'world', 'hello', 'python']

# join 拼接
```

（5）字符串的切片（列表，元组都支持，不支持字典）

```python
# 语法
# string[开始的索引:结束的索引:步长]
# 结束索引不包括该索引位置的字符
# 步长，默认就是1，逆序是-1
str = "0123456789"
# 截取 2-5 的字符串
print(str[2:6])
# 截取2到末尾的字符串
print(str[2:])
# 截取开始到5的字符串
print(str[:6])
# 截取完整的字符串
print(str[:])
# 从开始，每隔一个字符截取
print(str[::2])
# 截取末尾两个字符
print(str[-2:])
# 字符串逆序
print(str[::-1])
```

### 变量在内存中存储

```python
foo = [1,2]
foo1 = foo
foo.append(3)
# oo1 = foo为浅表复制，也称为浅拷贝，只是将foo1和foo指向相同存储地址，在foo中追加3后，二者都改变。 如果不想改变foo1，我们可使用deepcooy进行深拷贝
```

### 可变类型和不可变类型

数字、字符串和元组在内存中不可更改。

字典是可变类型，但是字典的key是不可变类型。

### 局部变量和全局变量

