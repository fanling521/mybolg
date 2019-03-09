# HTML教程

##  HTML简介

### 什么是HTML

- HTML 是用来描述网页的一种语言
- HTML 指的是超文本标记语言 (**H**yper **T**ext **M**arkup **L**anguage)
- HTML 标签是由**尖括号**包围的关键词，比如 `<html>` 
- HTML 标签通常是**成对出现**的，比如 `<b> `和 `</b>`

## HTML 基本结构

完整HTML包括 DOCTYPE声明、html根、head(title、mate等标签)、body等内容

### W3C标准和基本规范

w3c是指定html规范标准的机构。

- 确保所有的标签都使用小写字母
- 确保所有的属性值都放在引号里
- 确保所有成对标签出现的顺序、不成对的标签都用/>结束, ”/”和”>”之间不要有空格
-  声明你的编码语言，文档声明
- 为图片添加 alt 属性


## HTML基础语法
### HTML标签

1. 一级标题(headline) `<h1></h1>` ....`<h6></h6>`
2. 换行(blank row) `<br/>、<br>`
3. 段落()`<p></p>`
4. 水平线(horizontal rule)`<hr/>、<hr>`
5. 斜体 `<i></i>  <em></em>`
6. 加粗 `<b></b> <strong></strong>`
7. 图片 `<img/>`
   1. 属性`alt`：替换文本属性
   2. 属性`src`：图像的 URL 地址
      1. 相对路径
      2. 绝对路径  
   3. `width`,`height` 图片的宽度，高度
8. 超链接`<a></a>`
     1. 属性`href`：超链接指向的地址
     2. 属性`target`：打开方式，`_blank`新标签页打开，`_self`当前页面打开
     3. 锚链接：锚链接的标记地址`<a name="yourName"></a>`(也可以用id)，跳转`<a href="#yourName">跳转到yourName</a>`，将 # 符号和锚名称添加到 URL 的末端，可以跨页面跳转。
9. 无序列表 `<ul><li></li></ul>` 属性` type`
10. 有序列表`<ol><li></li></ol>` 属性 `type`、`start`
11. 定义列表`<dl><dt></dt><dd></dd></dl>`
12. 块状元素/行内（内联）元素
13. 表单
	1. `input` 常用type类型：`text,password,radio,checkbox(checked="checked"),file,hidden,select>option(selected="selected")`，此外还有`textarea`
	 2. 表单关联表元素`<label for="{id}"></label>`
	3. 表单元素的只读性：`readony`；禁用：`disabled`，表单的提交地址`action`，提交方式`method`(get|post)
	4. 按钮`button`、`submit`(提交)和`reset`(重置)
14. 表格`<table></table>`，**属性**：`align`(*水平位置center|left|right*)，`valign`(*垂直位置top|bottom|middle*)，`bgcolor`(*背景颜色*)，`cellspacing`(*单元格之间距离*)，`cellpadding`(*单元格和内容之间距离*)，`width/height`(*长宽*)，`border`(*边框1|0*)
	1. 标题：`<cption></caption>`
	2. 表头：`<thead><tr><th>...</th></tr></thead>`
	3. 主体：`<tbody><tr><td>...</td></tr></tbody>`
	4. 页脚：`<tfoot><tr><td>...</td></tr></tfoot>`
	5. 单元格属性：`colspan`（合并列），`colspan`（合并行）
15. `<iframe></iframe>`：**基础属性** `src`(目标源)，`width/height`：长宽，`scrolling`(滚动条auto|yes|no)，`frameborder`(边框)


### 特殊字符的转义

1. 空格(non-breaking space) `&nbsp;`
2. 大于(greater than) `&gt;`
3. 小于(less than) `&lt;`
4. 版权(copy right)`&copy;`
5. 引号(quotation marks)`&quot;`
6. & `&amp;`

### HTML例子

> 实例1-基本标签的使用 qingpingyue.html 


```html
<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
    <title>清平乐·年年雪里</title>
</head>
<body>
    <h1>清平乐·年年雪里</h1>
    朝代：<i>宋代</i>
    &nbsp;&nbsp;
    作者：<b>李清照</b>
    <hr>
    <p>年年雪里,常插梅花醉。</p>
    <p>挼尽梅花无好意,赢得满衣清泪。</p>
    <p>今年海角天涯,萧萧两鬓生华。</p>
    <p>看取晚来风势，故应难看梅花。</p>
</body>
</html>
```

> 实例2-表单样式yonghudendlu.html



```html
<!DOCTYPE html>
<html>

<head>
    <meta charset="UTF-8">
    <title>用户登录</title>
</head>

<body>
    <h1>用户登录</h1>
    <form action="login.html" method="post">
        用户名：<input type="text" name="userName" autocomplete="off">
        <br>
        密　码：<input type="password" name="password">
        <br>
        <input type="radio" name="version" value="jianyueban" checked="checked" id="jianyueban"><label for="jianyueban">简约版</label>
        <input type="radio" name="version" value="haohuaban" id="haohuaban"><label for="haohuaban">豪华版</label>
        <br>
        <input type="checkbox" name="rememberme" id="rememberme"><label for="rememberme">记住密码</label>
        <input type="checkbox" name="denglu" id="denglu"><label for="denglu">安全登录</label>
        <br>
        <input type="submit" value="登录"><input type="reset" value="重置">
    </form>
</body>

</html>
```

