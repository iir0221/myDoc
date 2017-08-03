# function
## 定义
### 函数声明定义
```js
function sum(num1,num2){
    return num1+num2;
}
```

```js
// 可以正常执行
alert(sum(10+10));
function sum(num1,num2){
    return num1+num2;
}
```
### 函数表达式定义
```js
var sum=function(num1,num2){
    return num1+num2;
};
```

```js
//报错
// unexpected identifier
alert(sum(10+10));
var sum=function(num1,num2){
    return num1+num2;
};
```
### 两者区别
除了什么时候可以通过变量访问函数这一点区别外，函数声明与函数表达式的语法其实是等价的

## 函数作为值
函数名本身就是变量名，所以函数可以当做值来用（例如：函数名可以作为参数传递，可以作为结果返回）。

### 函数作为参数传递

```js
// 函数作为参数传递
function callFunction(someFunction,someArgument){
    return(someFunction(someArgument));
}

function add10(num){
    return num+10;
}

// 要访问函数的指针而不执行函数，必须去掉函数名后面那对括号
var result=callFunction(add10,10);
```

### 函数作为其他函数的返回值
```js
//函数作为其他函数的返回值

function createComparisonFunction(propertyName) {
    return function (object1,object2) {
        var value1=object1[propertyName];
        var value2=object2[propertyName];

        if(value1<value2){
            return -1;
        } else if(value1>value2){
            return 1;
        } else {
            return 0;
        }
    };

}

var data=[{name:"zhang",age:27},{name:"wang",age:29}];
data.sort(createComparisonFunction("name"));
console.log(data[0].name);
data.sort(createComparisonFunction("age"));
console.log(data[0].name);
```

## 函数的属性
### arguments
 
 * 保存函数的参数
 * 拥有一个叫做callee的属性，指向arguments对象所在函数
 
 
 **arguments.callee属性的一个重要应用是在递归函数中，使函数名和函数执行可以解耦**
 
```js
// 函数名称和函数执行耦合
function factorial(num) {
    if (num<=1) {
        return 1;
    } else {
        return num*factorial(num-1);
    }
}

// arguments.callee在递归函数总的使用
function factorial2(num) {
    if (num<=1) {
        return 1
    } else {
        return num*arguments.callee(num-1);
    }
}

// 也可以通过命名函数表达式达成相同的效果
var factorial3=(
    function f(num) {
        if(num<=1) {
            return 1;
        } else {
            return num*f(num-1);
        }
    }
);

console.log(factorial(10));
console.log(factorial2(10));
console.log(factorial3(10));
```
### this
和java中的this类似，区别在于如果是全局函数，则this指向window对象
```js
window.color="red";
var o={color:"blue"};

function sayColor() {
    console.log(this.color);
}

sayColor(); //red
o.sayColor=sayColor;  //blue
o.sayColor();
```

### caller
```js
// caller属性指向调用当前函数的函数，如果在全局环境下使用当前函数，则返回null,否则返回函数
function outer() {
    inner();
}

function inner() {
    //耦合，不推荐
    console.log(inner.caller);
    // 推荐使用arguments.caller属性代替inner
    console.assert(arguments.callee.caller);
}

outer();
```

### length
函数的参数个数

### prototype

## 函数的方法

### apply()和call()

这两个方法只是调用参数的形式不一样，最重要的作用是扩展函数的作用域

####  扩展函数作用域
```js
window.color="red";
var o={color:"blue"};

function sayColor() {
console.log(this.color);
}

sayColor.call(this); //red
sayColor.call(window); //red
sayColor.call(o); //blue
```

#### 调用参数的形式
```js
// apply()和call()的调用方式
function sum(num1, num2) {
    return num1+num2;
}

// 将每个参数分别传入
function callSum(num1, num2) {
    return sum.call(this,num1,num2);
}

//第二个参数是arguments
function callSum2(num1, num2) {
    return sum.apply(this,arguments);
}

//第二个参数是数组或者Array的实例
function callSum3(num1, num2) {
    return sum.apply(this,[num1,num2]);
}

console.log(callSum(10,10));
console.log(callSum2(10,10));
console.log(callSum3(10,10));
```

### bind()
将this值绑定到传给bind()的值
```js
//将this值绑定到传给bind的值
window.color="red";

var o={color:"blue"};

function sayColor() {
    console.log(this.color);
}

//sayColor()调用bind()并传入对象o,创建objectSayColor函数，objectSayColor函数的this将等于o
var objectSayColor=sayColor.bind(o);
objectSayColor(); //blue
```

### 继承的函数toLocalString()，toString(),valueOf()

返回函数代码







