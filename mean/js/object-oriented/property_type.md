# Property Type
## Data Peroperties

```js
// 像这种有值的属性就是data properties
var p1={name:"zhang",age:12};
```
数据属性有4个描述其行为的特性：

* configurable 
* enumerable 
* writable  
* value 

对于直接在对象中定义的属性(如上例)，其configurable ，enumerable ，writable默认为true，value为指定的值。

**Object.definePropertity()** 方法也可以为对象添加数据属性，还可以修改数据属性的特性
```js
var person = {};
Object.defineProperty(person,"name",{
  value: "zhang"
});
// configurable ，enumerable ，writable默认为false
// Object {value: "zhang", writable: false, enumerable: false, configurable: false}
console.log(Object.getOwnPropertyDescriptor(person,"name"));
```

```js
var person={};
// 第一个参数为对象 第二个参数为对象属性 第三个参数为configurable，enumrable，writable，value中的一个或多个
Object.defineProperty(person,"name",{writable:false,value:"zhang"});
console.log(person.name);
person.name="wang";
// 输出zhang
console.log(person.name);
```
## Accessor Properties
**访问器属性是 ECMA 标准定义的属性类型，是用来实现 Javascript 引擎本身使用的，因此不建议开发者使用访问器属性。**

访问器属性不包含数据值，它们包含一对儿 getter 和 setter 函数（不过，这两个函数都不是必需的）。在读取访问器属性时，会调用 getter 函数，在写入访问器属性时，又会调用 setter 函数并传入新值。

访问器属性有4个描述其行为的特性：

* configurable 默认false
* enumerable  默认false
* get方法
* set方法

访问器属性与上述数据属性不同，不能直接在对象中定义，必须使用函数Object.defineProperty()定义
```js
Object.defineProperty(book, "year", {
    get:function () {
        console.log("getter 方法被调用")
        return this._year;
    },
    set:function (newValue) {
        console.log("setter 方法被调用")
        if(newValue>2017) {
            this._year=newValue;
            this.edition+=newValue-2017;
        }
    }
});
// 直接调用 book.year，即调用了这个访问器属性中定义的 get 方法，返回 book._year 这个数据属性。
// 如果给 book.year 赋值，就是调用了这个访问器属性中定义的 set 方法。
book.year=2018; //打印：setter 方法被调用
console.log(book.edition); //2
console.log(book.year); //打印：getter 方法被调用 2008
console.log(book._year); //2008
```
## 定义多个属性：Object.defineProperties()
```js
// Object.defineProperties() 一次定义多个属性
var book={};
Object.defineProperties(book,{
   _year:{
     value:2017  
   },
    edition:{
       value:1
    },
    year:{
       get:function () {
           return this._year;
       }, 
        set:function (newValue) {
           if(newValue>2017){
               this._year=newValue;
               this.edition+=newValue-2017;
           }
            
        }
    }
});

console.log(book.year);
```
## 读取属性的特性：Object.getOwnPropertyDescriptor
```js
var descriptor=Object.getOwnPropertyDescriptor(book,"_year");
console.log(descriptor.value);
console.log(descriptor.configurable);
console.log(typeof descriptor.get); //undefined
var descriptor=Object.getOwnPropertyDescriptor(book,"year");
console.log(descriptor.value); //undefined
console.log(descriptor.enumerable);
console.log(typeof descriptor.get); //function
```



## 方法总结
* Object.definePropertity()
* Object.defineProperties()
* Object.getOwnPropertyDescriptor()









