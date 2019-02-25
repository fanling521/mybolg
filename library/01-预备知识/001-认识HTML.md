# 认识 HTML

##  基本认识

### 什么是HTML

- HTML 是用来描述网页的一种语言
- HTML 指的是超文本标记语言 (**H**yper **T**ext **M**arkup **L**anguage)
- HTML 标签是由**尖括号**包围的关键词，比如 `<html>` 
- HTML 标签通常是**成对出现**的，比如 `<b> `和 `</b>`

## HTML 基本结构

完整HTML包括 html根、DOCTYPE声明、head(title、mate)、body等内容

### W3C标准和基本规范


## HTML语法
### 常见标签

1. 一级标题(headline) `<h1></h1>` ....`<h6></h6>`
2. 换行(blank row) `<br/>`
3. 段落()`<p></p>`
4. 水平线(horizontal rule)`<hr/>`
5. 斜体 `<i></i>  <em></em>`
6. 加粗 `<b></b> <strong></strong>`
7. 图片 `<img/>`
   1. 属性`alt`：替换文本属性
   2. 属性`src`：图像的 URL 地址
      1. 相对路径
      2. 绝对路径  
   3. `width`,`height` 图片的宽度，高度
8. 超链接`<a></a>`
	1. 属性`href`：
	2. 属性`target`：
### 特殊字符

1. 空格(non-breaking space) `&nbsp;`
2. 大于(greater than) `&gt;`
3. 小于(less than) `&lt`
4. 版权(copy right)`&copy;`
5. 引号(quotation marks)`&quot;`

### 实例

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

> 实例2-基本标签的使用 liqingzhao.html

```html
<!DOCTYPE html>
<html>
<head>
	<meta charset="UTF-8">
	<title>李清照</title>
</head>
<body>
	<h1>人物简介</h1>
	<p><b>李清照</b>（<i>1084年3月13日—约1155年</i>），号易安居士，汉族，山东省济南章丘人。宋代（南北宋之交）女词人，婉约词派代表，有“千古第一才女”之称。所作词，前期多写其悠闲生活，后期多悲叹身世，情调感伤。形式上善用白描手法，自辟途径，语言清丽。论词强调协律，崇尚典雅，提出词“别是一家”之说，反对以作诗文之法作词。能诗，留存不多，部分篇章感时咏史，情辞慷慨，与其词风不同。有《易安居士文集》《易安词》，已散佚。后人有《漱玉词》辑本。今有《李清照集校注》。</p>
	<hr>
	&copy;2019&nbsp;北风网版权所有
</body>
</html>
```

