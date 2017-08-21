# Form
## FormControls and FormGroups
* Encapsulate(封装) the inputs in our forms and give us objects to work with them
### FormControl
* A FormControl represents a single input field - it is the smallest unit of an Angular form.
### FormGroup
* A wrapper interface around a collection of FormControls

## FormsModuke and ReactiveFormsModule
> Remember that whenever we make directives available to our view, they will get attached to any element that matches their selector. (一旦将指令应用到视图中，它们就与匹配各自selector的元素关联)
### FormsModule
* give us template driven directives
    * ngModel
        * selector为ngModel
            > The NgModel directive specifies a selector of ngModel. 

            > This means we can attach it to our input tag by adding this sort of attribute: ngModel="whatever".

    * NgForm
        *  当将FormsModule导入后，NgForm 将自动关联所有的form
            > because of the default NgForm selector

        * 一个名为ngForm的FormGroup对象
        * 一个(ngSubmit)输出
### ReactiveFormsModule
* gives us directives
    * formControl
    * ngFormGroup

```html
<div class="ui raised segment">

  <h2 class="ui header">Demo Form:Sku</h2>
  <form #f="ngForm" (ngSubmit)="onSubmit(f.value)"
        class="ui form">
<!--
下面模板中添加的 ngModel 属性没有属性值，表示其属性值和 input 的 name 值相等，即也是 "sku"
当在 input 上加个一个无属性值的 ngModel 属性时，表示：
1. 建立一个单向数据绑定
2. 根据该 input 的 name 值，创建一个同名的 FormControl 对象
    由 ngModel 创建的 FormControl 对象会自动添加到父 FormGroup 对象中（本例中 f），
    并将 DOM 元素也和新建的该 FormControl 对象绑定。

-->
    <div class="field">
      <label for="skuInput">SKU</label>
      <input type="text"
        id="skuInput"
        placeholder="SKU"
        name="sku" ngModel>
    </div>
    <button type="submit" class="ui button">Submit</button>
  </form>
</div>
```
```ts
import { Component, OnInit } from '@angular/core';

@Component({
  selector: 'app-demo-form-sku',
  templateUrl: './demo-form-sku.component.html',
  styleUrls: ['./demo-form-sku.component.css']
})
export class DemoFormSkuComponent implements OnInit {

  constructor() { }

  ngOnInit() {
  }

  onSubmit(form:any):void {
    console.log('you submitted value:',form);
  }

}
```