********# 继承
依靠原型链支持实现继承

## 原型链
利用原型让一个引用类型继承另一个引用类型的属性和方法。
```js
function SuperType() {
    this.property=true;
}

SuperType.prototype.getSuperValue=function () {
    return this.property;
}

function SubType() {
    this.subProperty=false;
}

SubType.prototype=new SuperType();

SubType.prototype.getSubType=function () {
    return this.subProperty;
}

var instance=new SubType();
console.log(instance.getSuperValue()); //true

console.log(instance instanceof Object); //true
console.log(instance instanceof  SubType); //true
console.log(instance instanceof SuperType); //true
```
在上例中SubType继承自SuperType，SuperType继承自Object

原型链有两个主要问题

* 原型属性为引用类型值时，该属性被所有实例共享
* 创建子类型实例时，不能向父类的构造函数传递参数

## 借用构造函数
子类在创建实例时，可以向父类构造函数传递参数
```js
//借用构造函数
function SuperType2(name) {
    this.name=name;
}

function SubType2(name,age) {
    SuperType2.call(this,name);
    this.age=age;
}
var instance=new SubType2('zhang',21);
console.log(instance.name); //zhang
console.log(instance.age); //21
```


借用构造函数的问题
* 方法都在构造函数中定义，不能复用，父类的原型定义的方法对子类不可见。

## 组合继承：组合原型链和构造函数式继承
结合原型链和借用构造函数，避免它们的缺点，融合它们的有点，**是最常用的继承模式**

```js
// 组合继承
function SuperType3(name) {
    this.name=name;
    this.colors=['red','blue','green'];
}


SuperType3.prototype.sayName=function () {
    console.log(this.name);
};

function SubType3(name,age) {
    // 执行父类构造函数
    SuperType3.call(this,name);
    this.age=age;
}

// 父类对象作为子类原型
SubType3.prototype=new SuperType3();

SubType3.prototype.sayAge=function () {
    console.log(this.age);
};


var instance1=new SubType3('zhang',12);
instance1.colors.push('black');
console.log(instance1.colors); //[ 'red', 'blue', 'green', 'black' ]
instance1.sayName(); //zhang
instance1.sayAge(); //12

var instance2=new SubType3('wang',21);

console.log(instance2.colors); //['red','blue','green']
instance2.sayName(); //wang
instance2.sayAge(); //21
```
组合继承的问题：
* 调用父类构造函数两次，一次在创建子类型原型时，另一次在子类型构造函数内部。

## 原型式继承
把父类对象作为子类对象的原型，然后根据具体需求对子类对象进行增强。
同原型链一样，包含引用类型值的属性会被所有实例共享。

```js
// 原型式继承
function object(o) {
    function F() {

    }
    F.prototype=o; //父类对象作为子类原型
    return new F();
}

var p={
    name:'zhang',
    friends:['a','b','c']
};

var p2=object(p);
p2.name='wang';
p2.friends.push('d');

var p3=object(p);
p3.name='zhao';
p3.friends.push('e');

console.log(p.friends); //[ 'a', 'b', 'c', 'd', 'e' ]
```

ECMAScript5新增Object.create()方法规范化原型式继承,在传入一个参数情况下，该方法行为同上面定义的object一样。
```js
// Object.create()
var p4=Object.create(p);
p4.name='li';
p4.friends.push('f');

console.log(p4.name); //li
console.log(p4.friends); //[ 'a', 'b', 'c', 'd', 'e', 'f' ]
```

Object.create()方法第二个参数与Object.defineProperties()方法的第二个参数格式相同：每个属性都是通过自己的描述符定义的。以这种方式指定的任何属性都会覆盖原型对象上的同名属性。
```js

var p5=Object.create(p,{
    name:{
        value:'sun'
    }
});

console.log(p5.name); //sun
console.log(p5.friends); //[ 'a', 'b', 'c', 'd', 'e', 'f' ]
```
原型式继承问题：
* 同原型链一样，包含引用类型值的属性会被所有实例共享


## 寄生式继承——改进原型式继承
主要用于封装继承过程，将原型式继承和子类对象增强（如添加方法）的过程包装到一起。
```js
// 寄生式继承
function createPerson(original) {
    // 调用原型式继承
    var clone=object(original);
    // 子类对象增强
    clone.sayHi=function () {
        console.log('hi');
    };
    return clone;
}

var p6=createPerson(p);
p6.sayHi();
console.log(p6.name);
```
寄生式继承问题
* 同原型链一样，包含引用类型值的属性会被所有实例共享
* 子类对象增强不可复用


## 寄生组合式继承：引用类型最理想的继承范式
组合继承会调用父类的构造函数两次，如下例所示
```js
function SuperType4(name) {
    this.name=name;
    this.colors=['red','blue','green'];
}

SuperType4.prototype.sayName=function () {
    console.log(this.name);
};

function SubType4(name,age) {
    SuperType4.call(this,name); //第二次调用
    this.age=age;
}


SubType4.prototype=new SuperType4(); //第一次调用

SubType4.prototype.constructor=SubType4;
SubType4.prototype.sayAge=function () {
    console.log(this.age);
};
```

寄生组合式继承的思想是：**不必为了指定子类型的原型而调用父类型的构造函数，所需要的只是父类原型的一个副本。**本质上就是使用寄生式继承来继承父类的原型，并将结果指定给子类的原型。

```js
function inheritPrototype(subType, superType) {
    // 不同于原型式继承，仅仅子类对象的原型不是父类对象而是父类对象的原型
    var prototype=object(superType.prototype); //创建对象
    prototype.constructor=subType; //增强对象
    subType.prototype=prototype;  //指定对象
}

function SuperType5(name) {
    this.name=name;
    this.colors=['red','blue','green'];
}

SuperType5.prototype.sayName=function () {
    console.log(this.name);
};

function SubType5(name,age) {
    // 可避免包含引用类型值的属性被所有实例共享
    SuperType5.call(this,name); // 调用一次
    this.age=age;
}

// 可避免二次调用父类构造函数
inheritPrototype(SubType5,SuperType5);
SubType5.prototype.sayAge=function () {
    console.log(this.age);
};

var p7=new SubType5('lin',23);
p7.sayName(); //lin
```






## 方法属性总结

* Object.create()
 