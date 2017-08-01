# wrapping up
```
1. Split your app into components 
2. Create the views
3. Define your models
4. Display your models
5. Add interaction
```


# Env
* nodejs and npm
* brew

## Typescript
### Install 
``` bash
npm install -g typescript
```
## Angular CLI
### Install 
```bash
brew install watchman
```
```bash
npm install -g @angular/cli@1.0.0
```
## Other Util
### tree
#### Install 
```bash
brew install tree
```
#### Usage
```bash
tree -F -L 1
```
```
xinyuazh-mac:angular-hello-world xinyuan.zhang$ tree -F -L 1
.
├── README.md
├── e2e/
├── karma.conf.js
├── node_modules/
├── package-lock.json
├── package.json
├── protractor.conf.js
├── src/
├── tsconfig.json
└── tslint.json

3 directories, 7 files

```

# Angular CLI
```bash
ng --help
```
## New project
```bash
ng new --ng4 angular-hello-world
```
## Start http serve
### Default Port 4200
```bash
ng serve
```
```
ng serve --port 9001
```
## New component
```bash
ng generate component hello-world
```

# Component
* a basic component has two parts
    * A Component decorator
    * A component definition class
```
ng generate component hello-world

├── app
│   ├── app.component.css
│   ├── app.component.html
│   ├── app.component.spec.ts
│   ├── app.component.ts
│   ├── app.module.ts
│   └── hello-word
│       ├── hello-word.component.css
│       ├── hello-word.component.html
│       ├── hello-word.component.spec.ts
│       └── hello-word.component.ts
├── assets
├── environments
│   ├── environment.prod.ts
│   └── environment.ts
├── favicon.ico
├── index.html
├── main.ts
├── polyfills.ts
├── styles.css
├── test.ts
├── tsconfig.app.json
├── tsconfig.spec.json
└── typings.d.ts
```
## @Component
### @Component 声明
```ts
import { Component, OnInit } from '@angular/core';
@Component({
    selector: 'app-hello-word',
    templateUrl: './hello-word.component.html',
    styleUrls: ['./hello-word.component.css']
})

export class HelloWordComponent implements OnInit {
    constructor() { }
    ngOnInit() {
    }
}
```
* selector
    * 指定Component的标签名称
* templateUrl
    * 指定模板
    * 可替换为如下方式
        ```ts
        @Component({
            selector: 'app-hello-word',
            template: `
                <p>
                    hello-world works inline!
                </p>
            `
        ```
* styleUrls 
    * 指定css style
    * key是一个数组，因为可以指定多个css文件

### Component Class
```ts
import { Component, OnInit } from '@angular/core';

@Component({
    selector: 'app-user-item',
    templateUrl: './user-item.component.html',
    styleUrls: ['./user-item.component.css']
})

export class UserItemComponent implements OnInit {
    name:string;
    constructor() {
        this.name = 'Felipe';
    }
    ngOnInit() {
    }
}
```
* property
    * name:string
* constructor
    * this.name = 'Felipe'

* Arrays
    ```ts
    import { Component, OnInit } from '@angular/core';

    @Component({
        selector: 'app-user-list',
        templateUrl: './user-list.component.html',
        styleUrls: ['./user-list.component.css']
    })

    export class UserListComponent implements OnInit {
        names:string[];
        constructor() {
            this.names = ['Ari','Carlos','Felipe','Nate'];
        }
        ngOnInit() {
        }
    }
    ```
    ```html
    <ul>
        <li *ngFor="let name of names">Hello {{ name }}</li>
    </ul>
    ```
## @Input
```ts
import { Component, OnInit, Input } from '@angular/core';

@Component({
    selector: 'app-user-item',
    templateUrl: './user-item.component.html',
    styleUrls: ['./user-item.component.css']
})

export class UserItemComponent implements OnInit {
    @Input() name:string;

    constructor() {
    }
    ngOnInit() {
    }
}
```
* pass value from parent to input
    ```html
    <ul>
        <li *ngFor="let individualUserName of names">
            <app-user-item [name]="individualUserName"></app-user-item>
        </li>
    </ul>
    ```

<span id="hostBinding"></span>
## @ HostBinding()
* 为host element 设置属性值，例如class
* 使Component封装性更好
*   ```ts
    export class ArticleComponent implements OnInit {
    @HostBinding('attr.class') cssClass = 'row';
    votes:number;
    title:string;
    link:string;

    ···
    ```
# Class
## 参数可选
* 使用?标识
    ```ts
    export class Article {
    title: string;
    link: string;
    votes: number;

    constructor(title: string, link: string, votes?: number) {
        this.title = title;
        this.link = link;
        this.votes = votes || 0;
    }
    }
    ```
# MVC
## M和C有同样的方法
* 处理Component view相关业务
    * article.component.ts
        ```ts
        voteUp(): boolean {
            this.article.voteUp();
            return false;
        }
    ```
* 处理Model相关变化
    * article.module.ts
        ```ts
        voteUp(): void {
            this.votes += 1;
        }
        ```

# Bootstrap
```
ng serve
    |_ look for angular.cli.json
        |_ "main" specifies main file
            |_ in this case is "main.ts"
            |_ main.ts bootstrap a module
                |_ platformBrowserDynamic().bootstrapModule(AppModule);
                    |_APPModule
                        |_ @NgModule({
                                declarations: [
                                    AppComponent,
                                    HelloWordComponent,
                                    UserItemComponent,
                                    UserListComponent
                                ],
                                imports: [
                                    BrowserModule
                                ],
                                providers: [],
                                bootstrap: [AppComponent]
                        })
                        |_ AppModule bootstrap the App
                            |_ bootstrap: [AppComponent] 
                        |_ AppModule specifies the top-level component
                            |_ in this case is AppComponent
                            |_ Appcomponent has <app-user-list> which define by ourself
```
## @NgModule
* declarations
    * specifies the components that are defined in this module
* imports
    * dependencies
* providers
    * used for DI
* bootstrap
    * 当这个module被用来bootstrap app,指定top-level Component

# Form
```html
<form class="ui large form segment">
  <h3 class="ui header">Add a Link</h3>
  <div class="field">
    <label for="title">Title:</label>
    <input name="title" #newtitle>
  </div>
  <div class="field">
    <label for="link">Link:</label>
    <input name="link" #newlink>
  </div>
  <button (click)="addArticle(newtitle,newlink)" class="ui positive right floated button">
    Submit link
  </button>
</form>
```
## Binding inputs to values
* #value代表变量可以通过表达式解析
* 本例，#newtitle 是一个Object，代表input DOM,实际类型是HTMLInputElement，newtitle.value是input value
## Binding actions to events
* 点击事件
    * (click)="addArticle(newtitle,newlink)" class="ui positive right floated button"
    * addArticle定义在class中
        *  ```ts
            addArticle(title:HTMLInputElement,link:HTMLInputElement):boolean {
                console.log(`Adding article title: ${title.value} and link: ${link.value}`);
                return false;
            }
            ```
            * 使用反引号\`,反引号\`会解析表达式

# Deployment
## building
```bash
ng build --target=production --base-href '/'
```
* base-href
    * 指定root URL
## uploading
[now](https://zeit.co/)




# JavaScript
## href click event return false
可以禁止点击链接跳转的动作
```html
<li class="item">
    <a href (click)="voteDown()">
    <i class="arrow down icon"></i>
    downvote
    </a>
</li>
```
```ts
voteDown(): boolean {
    this.votes -=1;
    return false;
}
```

## Arrow functions
[教程](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/Arrow_functions)

### forEach
.forEach is a method on Array that accepts a function as an argument and calls that function for each element in the Array.
```ts
class Report {
    data:Array<string>;
    constructor(data:Array<string>) {
        this.data = data;
    }
    run() {
        this.data.forEach(function(line){console.log(line);});
    }
}
```