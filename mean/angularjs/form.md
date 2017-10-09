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
        * input -- NgModel
        * selector为ngModel
            > The NgModel directive specifies a selector of ngModel. 

            > This means we can attach it to our input tag by adding this sort of attribute: ngModel="whatever".

    * NgForm
        * form -- NgForm
        *  当将FormsModule导入后，NgForm 将自动关联所有的form
            > because of the default NgForm selector

        * 一个名为ngForm的FormGroup对象
        * 一个(ngSubmit)输出
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

```html
<div class="ui raised segment">

  <h2 class="ui header">Demo Form:Sku</h2>
  <!--a FormGroup named ngForm-->
  <!--  NgForm is automatically attached to <form> tags
  (because of the default NgForm selector),
  which means we don’t have to add an ngForm attribute to use NgForm -->
  <!--#f=ngForm create a local variable for this view-->


  <!--ngForm:FormGroup-->
  <form #f="ngForm" (ngSubmit)="onSubmit(f.value)"
        class="ui form">

    <div class="field">
      <label for="skuInput">SKU</label>
      <!--sku:FormControl(A FormControl represents a single input field-it is the smallest unit of an Angular form)-->
      <!--ngModel:a one-way data binding, a formControl automatically added to the parent FormGroup named sku-->
      <input type="text"
        id="skuInput"
        placeholder="SKU"
        name="sku" ngModel>
    </div>
    <button type="submit" class="ui button">Submit</button>
  </form>
</div>
```
### ReactiveFormsModule
* gives us directives
    * formControl
    * ngFormGroup
### FormBuilder
更灵活的方
* control - creates a new FormControl
* group - creates a new FormGroup式
```ts
import { Component, OnInit } from '@angular/core';
import {
  FormBuilder,
  FormGroup
} from '@angular/forms';

@Component({
  selector: 'app-demo-form-sku-with-builder',
  templateUrl: './demo-form-sku-with-builder.component.html',
  styleUrls: ['./demo-form-sku-with-builder.component.css']
})
export class DemoFormSkuWithBuilderComponent implements OnInit {

  myForm:FormGroup;

  constructor(fb:FormBuilder) {
    this.myForm = fb.group({
      'sku':['ABC123']
    });
  }

  ngOnInit() {
  }

  onSubmit(value:string):void {
    console.log('you submitted value:',value);
  }

}
```
```html
<div class="ui raised segment">
  <h2 class="ui header">Demo Form: Sku with Builder</h2>
  <!--when we have an existing FormGroup: we use formGroup like this:-->
  <form [formGroup]="myForm" (ngSubmit)="onSubmit(myForm.value)" class="ui form">
    <div class="field">
      <label for="skuInput">SKU</label>

      <!--When we want to bind an existing FormControl to an input we use formControl:-->
      <input type="text" id="skuInput"
             placeholder="SKU"
             [formControl]="myForm.controls['sku']">
    </div>
    <button type="submit" class="ui button">Submit</button>
  </form>
</div>
```

## validator
To use validators we need to do two things:
* Assign a validator to the FormControl object
* Check the status of the validator in the view and take action accordingly

There are two ways we can access the validation value in the view:
* We can explicitly assign the FormControl sku to an instance variable of the class - which is more verbose, but gives us easy access to the FormControl in the view.
* We can lookup the FormControl sku from myForm in the view. This requires less work in the component definition class, but is slightly more verbose in the view.



