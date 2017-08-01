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
* ()=>{};
    * ES5
        ```js
        // ES5-like example
        var data = ['Alice Green', 'Paul Pfifer', 'Louis Blakenship'];
        data.forEach(function(line) { console.log(line); });
        ```
    * TypeScript
        ```ts
        // Typescript example
        var data: string[] = ['Alice Green', 'Paul Pfifer', 'Louis Blakenship']; 
        data.forEach( (line) => console.log(line) );
        ```
* share the same this as the surrounding code
    * ES5
    ```js
    var nate={
        name: "Nate",
        guitars: ["Gibson", "Martin", "Taylor"], 
        printGuitars: function() {
            var self = this;
            this.guitars.forEach(function(g) {
                // this.name is undefined so we have to use self.name
                console.log(self.name + " plays a " + g);
            });
        } 
    };
    ```
    * TypeScript
    ```ts
    var nate = {
        name:"Nate",
        guitars:["Gibson","Martin","Taylor"],
        printGuitars:function() {
            this.guitars.forEach((g)=>{
                console.log(this.name+" plays a "+ g);
            });
        }
    };
    ```
### template strings  ``
* Variables within strings (without being forced to concatenate with +) 
    ```ts
    var firstName = "Nate";
    var lastName = "Murray";
    // interpolate a string
    var greeting = `Hello ${firstName} ${lastName}`;
    console.log(greeting);
    ```
* Multi-line strings
    ```ts
    var template = `
    <div>
    <h1>Hello</h1>
    <p>This is a great website</p>
    </div>
    `
    ```
