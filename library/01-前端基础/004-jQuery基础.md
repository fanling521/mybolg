# jQuery 基础

## 什么是jQuery

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

