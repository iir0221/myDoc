# DOM


window对象对应者**浏览器窗口本身**，这个对象的属性和方法通常称为BOM，其提供了window.open，window.blur等方法。

所有 JavaScript 全局对象、函数以及变量均自动成为 window 对象的成员。

甚至 HTML DOM 的 document 也是 window 对象的属性之一：
```js
window.document.getElementById("header");
```
与此相同：
```js
document.getElementById("header");
```

## 节点
### 元素节点
所有的tag，html是跟元素节点

### 文本节点
元素节点中包含的文本称为文本节点

### 属性节点
通过元素节点的属性表示的节点

## 属性
### class属性
```html
<p class="special">class属性</p>
<h2 class="special">class属性</h2>
```

为所有class属性为special的元素定义样式
```css
.special {
  font-style:italic;
}
```
指定为某个class为special的元素定义样式
```css
h2.special {
  font-style:italic;
}
```
一个元素可以有多个class
```html
<html>
<head>
<style type="text/css">
h1.intro
{
color:blue;
text-align:center;
}
.important {background-color:yellow;}
</style>
</head>

<body>
<h1 class="intro important">Header 1</h1>
<p>A paragraph.</p>
</body>

</html>
```
### id属性
唯一的
```html
<ul id="id1"></ul>
```
为id为id1的元素定义样式
```css
#id1 {
  font-style:italic;
}
```
为包含在id为id1的元素里的其他元素定义样式
```css
#id1 li {
  font-style:italic;
}
```
## 获取元素节点
```html
var food = document.getElementById("food") <!--返回一个对象-->
<!--document下所有的li-->
ducument.getElementsByTagName(“li”)<!--返回一个对象数组-->
<!--food下所有的li-->
food.getElementsByTagName(“li”)<!--返回一个对象数组-->
<!--html5 dom新增-->
document.getElementsByClassName("sale")<!--返回一个对象数组-->
food.getElementsByClassName(“sale”)<!--返回一个对象数组-->
```
## 获取设置属性
```html
object.getAttribute(attribute)
object.setAttribute(attribute,value)
```
这两个方法不属于document对象，只能通过元素节点调用。

通过setAttribute做出的修改不会反映在文档本身的源代码里，DOM先加载静态内容，再动态刷新，动态刷新不影响文档的静态内容。