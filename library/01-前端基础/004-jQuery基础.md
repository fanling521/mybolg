# jQuery 基础

## 什么是jQuery

jQuery 是一个 JavaScript 库。

jQuery 极大地简化了 JavaScript 编程。

## jQuery入口函数

```javascript
$(document).ready(function(){
    ....
    ...
});
//缩写
$(function(){
    ...
    ...
});
```



## jQuery实例

### 设置样式

**CSS代码**

```css
.border{
	border: 1px solid #ccc;
}
.height{
	height: 100px;
}
```

**jQuery代码**

```javascript
$(function() {
	$("img").click(function(){
		//第一种 css()
		$(this).css("border","1px solid #ccc");
		//第二种 addClass()
		$(this). ('border').addClass('height');
	});
});
```

**隐式迭代**：

`$('div') `这其实是一个数组集合，并不是直接的DOM元素，**设置值**的时候无需再遍历即可全部赋值。

```javascript
$(function() {
	//原生js
	var divs =document.getElementsByTagName("div");
	for(var i =0;i<divs.length;i++){
		divs[i].innerHTML="使用原生js设置值";
	}
	//使用jQuery
	$("div").html("使用jQuery设置值");
});
```

### Dom和jQuery对象的转化

**说明**：两者对象必须使用各自的方法和属性。

jQuery对象是一个DOM元素数组。

通过jQuery对象的下标获取的对象的都是DOM对象

```javascript
//DOM对象转jQuery对象
var div=document.getElementById("divid");
//转化
var $div=$(div);
$div.html("DOM对象转jQuery对象");
//jQuery对象转DOM对象，用下标的方式
var $div2=$("div");
//取第二个div
var div2=$div2[1];
// var div2=$div2.get(1);
div2.innerHTML="jQuery对象转DOM对象，用下标的方式";
```

### jQuery选择器

####  基本选择器

按照CSS选择器来：交集选择器，并集选择器，后台选择器，子代选择器

####  层次选择器

| 名称       | 用法                                    |
| ---------- | --------------------------------------- |
| 后代选择器 | $(#menu span)选择#menu元素下的span元素  |
| 子代选择器 | $(#menu>span) 选择#menu的所有span子元素 |
| 相邻选择器 | $(h2+dl)选取h2相邻的一个元素dl          |
| 兄弟选择器 | $(h2~dl)选择h2后面的所有dl元素          |

####  属性选择器

| 名称                     | 用法                  |
| ------------------------ | --------------------- |
| $("li[title='item']")    | li属性title等于item   |
| $("li[title^='aa']")     | li属性title从aa开始的 |
| $("li[title&amp;='bb']") | li属性title以bb结尾的 |
| $("li[title*='cc']")     | li属性title包含cc的   |

####  过滤选择器

| 名称          | 用法                        |
| ------------- | --------------------------- |
| :first        | 选取第一个元素              |
| :last         | 选取最后一个元素            |
| :even         | 选取索引是偶数的所有元素    |
| :odd          | 选取索引是奇数的所有元素    |
| :eq(index)    | 选取等于index的索引所有元素 |
| :gt(index)    | 选取大于index的索引所有元素 |
| :lt(index)    | 选取小于index的索引所有元素 |
| :not(seletor) | 选取不是seletor的所有元素   |
| :header       | 选取所有headline元素        |
| :focus        | 选取获取焦点的元素          |
| :hidden       | 选取所有隐藏的元素          |
| :visible      | 选取所有可见的元素          |

#### 内容选择器

| 名称            | 用法                            |
| --------------- | ------------------------------- |
| :contains('xx') | 选取所有包含xx文本的元素        |
| :has(seletor)   | 选取所有含有seletor元素的元素   |
| :empty          | 选取所有不含空文本/空元素的元素 |
| :parent         | 选取所有包含子元素或文本的元素  |

#### 表单选择器

使用的是`:[type]`，不再说明，如`:text`，更多的时候，`:input`选取的是全部表单元素。

## HTML页面操作

```javascript
// js innerHTML 获取元素中的html代码
var html_1=obj.innerHTML;
// 对应的jQuery方法为
var html_2=$obj.html();
// js innerText 获取元素中的文本内容
var text_1=obj.innerText;
// 对于的jQuery方法为
var text_2=$obj.text();
// 表单中的获取值的方法是
// js
var val_1=document.getElementById("#text").value;
// jQuery
var val=$("input[type='text']").val();
```

## 节点操作

```javascript
// 选取元素div追加内容
$("#div").append("<p>hello</p>")
// 移除选取的元素
$("#div span").remove();
// 选取相邻之后的元素
$("#div").next().css("color","red");
// 选取相邻之前的元素
$("#div").prev().css("color","blue");
// 选取父元素
$("#div").parent().css("color","blue");
// 选取指定的父元素
$("#div").parents("tr").css("color","blue");
// 选取元素前后所有同辈元素
$("#div").siblings().css("color","blue");
```

## 事件委托

事件委托是利用事件冒泡，只指定一个事件处理程序来管理某一类型的所有事件。

在JavaScript中添加到页面上的事件处理程序的个数直接关系到页面的整体运行性能。为什么呢？因为，每个事件处理函数都是对象，对象会占用内存，内存中的对象越多，性能就越差。此外，必须事先指定所有的事件处理程序而导致的DOM访问次数，会延迟整个页面的交互就绪时间。

等待加载完毕后，再添加事件。

```javascript
// 渲染完毕后新增的元素需要添加点击事件
// jQuery
$("table").on("click", ".del",  function() {
	$(this).parents("tr").remove();
});
// js 是 addEventListener
```

