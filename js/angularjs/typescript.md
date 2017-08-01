# TypeScript
## Types
### TSUN

## Classes
* ES5
    * object-oriented programming (OOP) 
        * [Object-Oriented Js](https://developer.mozilla.org/en-US/docs/Learn/JavaScript/Objects)
    * inheritance 
        * [Inheritance and the prototype chain](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Inheritance_and_the_prototype_chain)
* TypeScript
    * Classes
        * TypeScript 类只能有一个constructor
        * ES6 类可以有多个constructor
    * inheritance
        * key word:extends
## Decorators
## Imports
## Language Utilities
### fat arrow function syntax
用于简写函数
* ES5
    ```js
    // ES5-like example
    var data = ['Alice Green', 'Paul Pfifer', 'Louis Blakenship'];
    data.forEach(function(line) { console.log(line); });
    ```
* TypeScript
    ```ts
    // Typescript example
    var data: string[] = ['Alice Green', 'Paul Pfifer', 'Louis Blakenship']; data.forEach( (line) => console.log(line) );
    ```
### template strings