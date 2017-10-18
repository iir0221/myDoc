# DI
## manually use the injector
```ts
@Injectable()
export class UserService {

  user:any;

  setUser(newUser) {
    this.user = newUser;
  }

  getUser():any {
    return this.user;
  }
}
```

```ts
export class UserDemoComponent implements OnInit {

  userName:string;
  userService:UserService;

  constructor() {
    const injector:any = ReflectiveInjector.resolveAndCreate([UserService]);
    this.userService = injector.get(UserService);
  }

  signIn():void {
    this.userService.setUser({
      name:'Nate Murray'
    });

    this.userName = this.userService.getUser().name;
    console.log('User name is:',this.userName);
  }

  ngOnInit() {
  }

}
```

```html
<div>

  <p *ngIf="userName"
  class="welcome">
    Welcome:{{userName}}
  </p>
  <button (click)="signIn()"
  class="ui button">
    Sign In
  </button>

</div>
```
In our component’s constructor we are using a static method from <b>ReflectiveInjector</b> called <b>resolveAndCreate</b>. That method is responsible for creating a new injector. The parameter we pass in is an array with all the injectable things we want this new injector to know. In our case, we just wanted it to know about the UserService injectable.

## Providing Dependencies with NgModule

app.module.ts
```ts
import { BrowserModule } from '@angular/platform-browser';
import { NgModule } from '@angular/core';

import { AppComponent } from './app.component';
import { UserDemoComponent } from './user-demo/user-demo.component';
import {UserDemoModule} from "./user-demo/user-demo.module";

@NgModule({
  declarations: [
    AppComponent,
    UserDemoComponent
  ],
  imports: [
    BrowserModule,
    UserDemoModule
  ],
  providers: [],
  bootstrap: [AppComponent]
})
export class AppModule { }
```
user.service.ts
```ts
import {Injectable} from "@angular/core";
/**
 * Created by xinyuan.zhang on 10/12/17.
 */
@Injectable()
export class UserService {

  user:any;

  setUser(newUser) {
    this.user = newUser;
  }

  getUser():any {
    return this.user;
  }
}
```
user-demo.module.ts
```ts
import {CommonModule} from "@angular/common";
import {UserService} from "../service/user.service";
import {NgModule} from "@angular/core";
@NgModule({
  imports:[
    CommonModule
  ],
  providers:[
    UserService
  ],
  declarations:[]
})

export class UserDemoModule{}
```
```ts
import {Component, OnInit, ReflectiveInjector} from '@angular/core';
import {UserService} from "../service/user.service";

@Component({
  selector: 'app-user-demo',
  templateUrl: './user-demo.component.html',
  styleUrls: ['./user-demo.component.css']
})
export class UserDemoComponent implements OnInit {

  userName:string;

  // When this component is created on our page Angular will resolve and
  // inject the UserService singleton.
  constructor(private userService:UserService) {

  }

  signIn():void {
    this.userService.setUser({
      name:'Nate Murray'
    });

    this.userName = this.userService.getUser().name;
    console.log('User name is:',this.userName);

  }

  ngOnInit() {
  }

}
```
```html
<div>

  <p *ngIf="userName"
  class="welcome">
    Welcome:{{userName}}
  </p>
  <button (click)="signIn()"
  class="ui button">
    Sign In
  </button>

</div>
```