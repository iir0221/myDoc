# syntax

## 数组和对象

var a = \[\];  //数组

var b = {};  //对象

### 内置对象\(JS内置\)

Object

Function

Array

String

Boolean

Number

Date

RegExp

Error

EvalError

RangeError

ReferenceError

SyntaxError

TypeError

URIError

### 宿主对象\(浏览器内置\)

所有非本地对象都是宿主对象（host object），即由 ECMAScript 实现的宿主环境提供的对象。

**具体到web应用，由浏览器提供的域定义对象被称为宿主对象。**

**所有 BOM 和 DOM 对象都是宿主对象。**

### Reference

[http://www.w3school.com.cn/js/pro\_js\_object\_types.asp](http://www.w3school.com.cn/js/pro_js_object_types.asp)

## 变量

### 变量类型

* Number（primitive,wrapper）

* String\(primitive,wrapper\)

* Boolean\(primitive,wrapper\)

* Symbol \(new in ECMAScript 6\)

* Object:

  * Function

  * Array

  * Date

  * RegExp

* null

* undefined


### 其他

NaN

The **NaN** value is a special value that indicates that the entity is not a number.

* NaN != NaN 
* isNaN\(\)

Infinity

* Number.POSITIVE\_INFINITY
* Number.NEGATIVE\_INFINITY
* isInfinite\(\)

### 变量作用域

**使用某个变量前最好用var声明变量（否则为**implicit globals**）**，尤其对于函数中的局部变量。

如果在某个函数中使用了var，那个变量就将被视为一个局部变量，它只存在于这个函数的上下文中；反之，如果没有使用var，那个变量就将被视为一个全局变量，如果脚本里已经存在一个与之同名的全局变量，这个函数就会改变那个全局变量的值。

例：

```js
function square(num) {
  total = num*num;
  return total;
}

var total = 50;
var number = square(20);
alert(total);
```

此时全局变量total的值被改变为400。

函数在行为方面应该像一个自给自足的脚本，定义一个函数时，一定要把它内部的变量全部明确的声明为局部变量。如果总是在函数中使用var来定义变量，就能避免任何形式的二义性隐患。

## 异步

注意,在浏览器中有几种特殊情况会“阻塞”程序执行,并且通常我们会建议你不要使用它们: alert、 prompt、 confirm  
和同步XHR。

