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

first
```ts
export class DemoFormWithValidationsExplicitComponent implements OnInit {

  myForm:FormGroup;
  sku:AbstractControl;
  // To assign a validator to a FormControl object
  // we simply pass it as the second argument to our FormControl constructor:
  constructor(fb:FormBuilder) {
    this.myForm = fb.group({
      'sku':['',Validators.required]
    });
    this.sku=this.myForm.controls['sku'];
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
  <h2 class="ui header">Demo Form: with validations (explicit)</h2>
  <form [formGroup]="myForm"
        (ngSubmit)="onSubmit(myForm.value)"
        class="ui form"
        [class.error]="!myForm.valid && myForm.touched">
    <div class="field"
         [class.error]="!sku.valid && sku.touched">
      <label for="skuInput">SKU</label>

      <input type="text"
             id="skuInput"
             placeholder="SKU"
             [formControl]="sku">

      <div *ngIf="!sku.valid"
           class="ui error message">SKU is invalid
      </div>
      <!--To look up a specific validation failure we use the hasError method:-->
      <div *ngIf="sku.hasError('required')"
           class="ui error message">SKU is required
      </div>
    </div>
    <!--a FormGroup is valid if all of the children FormControls are also valid.-->
    <div *ngIf="!myForm.valid"
         class="ui error message">Form is invalid
    </div>
    <button type="submit" class="ui button">Submit</button>
  </form>
</div>
```
second
```ts
export class DemoFormWithValidationsShorthandComponent implements OnInit {

  myForm:FormGroup;
  // To assign a validator to a FormControl object
  // we simply pass it as the second argument to our FormControl constructor:
  constructor(fb:FormBuilder) {
    this.myForm = fb.group({
      'sku':['',Validators.required]
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
  <h2 class="ui header">Demo Form: with validations (explicit)</h2>
  <form [formGroup]="myForm"
        (ngSubmit)="onSubmit(myForm.value)"
        class="ui form"
        [class.error]="!myForm.valid && myForm.touched">
    <div class="field"
         [class.error]="!myForm.controls['sku'].valid && myForm.controls['sku'].touched">
      <label for="skuInput">SKU</label>

      <input type="text"
             id="skuInput"
             placeholder="SKU"
             [formControl]="myForm.controls['sku']">

      <div *ngIf="!myForm.controls['sku'].valid"
           class="ui error message">SKU is invalid
      </div>
      <!--To look up a specific validation failure we use the hasError method:-->
      <div *ngIf="myForm.controls['sku'].hasError('required')"
           class="ui error message">SKU is required
      </div>
    </div>
    <!--a FormGroup is valid if all of the children FormControls are also valid.-->
    <div *ngIf="!myForm.valid"
         class="ui error message">Form is invalid
    </div>
    <button type="submit" class="ui button">Submit</button>
  </form>
</div>
```
### custom validation
```ts
function skuValidator(control: FormControl): { [s: string]: boolean } {
  if (!control.value.match(/^123/)) {
    return {invalidSku: true};
  }
}

@Component({
  selector: 'app-demo-form-with-custom-validation',
  templateUrl: './demo-form-with-custom-validation.component.html',
  styleUrls: ['./demo-form-with-custom-validation.component.css']
})
export class DemoFormWithCustomValidationComponent implements OnInit {

  myForm:FormGroup;
  sku: AbstractControl;

  // Validators.compose wraps our two validators and lets us assign them both to the FormControl.
  // The FormControl is not valid unless both validations are valid
  constructor(fb: FormBuilder) {
    this.myForm = fb.group({
      'sku': ['', Validators.compose([
        Validators.required, skuValidator])]
    });

    this.sku = this.myForm.controls['sku'];
  }

  onSubmit(value: string): void {
    console.log('you submitted value: ', value);
  }

  ngOnInit() {
  }

}
```
```html
<div class="ui raised segment">
  <h2 class="ui header">Demo Form: with custom validations</h2>
  <form [formGroup]="myForm"
        (ngSubmit)="onSubmit(myForm.value)"
        class="ui form"
        [class.error]="!myForm.valid && myForm.touched">

    <div class="field"
         [class.error]="!sku.valid && sku.touched">
      <label for="skuInput">SKU</label>
      <input type="text"
             id="skuInput"
             placeholder="SKU"
             [formControl]="sku">
      <div *ngIf="!sku.valid"
           class="ui error message">SKU is invalid</div>
      <div *ngIf="sku.hasError('required')"
           class="ui error message">SKU is required</div>
      <div *ngIf="sku.hasError('invalidSku')"
           class="ui error message">SKU must begin with <span>123</span></div>
    </div>

    <div *ngIf="!myForm.valid"
         class="ui error message">Form is invalid</div>
    <button type="submit" class="ui button">Submit</button>
  </form>
</div>
```
## watch change
* To watch for changes on a control we:
 * get access to the EventEmitter by calling control.valueChanges.Then we 
 * add an observer using the .subscribe method
 ```ts
export class DemoFormWithEventsComponent implements OnInit {

  myForm:FormGroup;
  sku:AbstractControl;


  constructor(fb:FormBuilder) {
    this.myForm = fb.group({
      'sku':['',Validators.required]
    });

    this.sku = this.myForm.controls['sku'];

    this.sku.valueChanges.subscribe(
      (value:string)=>{
        console.log('sku changed to:',value);
      }
    );

    this.myForm.valueChanges.subscribe((
      form:any)=>{
        console.log('form changed to:',form)
      }
    );
  }

  ngOnInit() {
  }

  onSubmit(form: any): void {
    console.log('you submitted value:', form.sku);
  }

}
 ```
 ```html
<div class="ui raised segment">
  <h2 class="ui header">Demo Form: with events</h2>
  <form [formGroup]="myForm"
        (ngSubmit)="onSubmit(myForm.value)"
        class="ui form">

    <div class="field"
         [class.error]="!sku.valid && sku.touched">
      <label for="skuInput">SKU</label>
      <input type="text"
             class="form-control"
             id="skuInput"
             placeholder="SKU"
             [formControl]="sku">
      <div *ngIf="!sku.valid"
           class="ui error message">SKU is invalid</div>
      <div *ngIf="sku.hasError('required')"
           class="ui error message">SKU is required</div>
    </div>

    <div *ngIf="!myForm.valid"
         class="ui error message">Form is invalid</div>

    <button type="submit" class="ui button">Submit</button>
  </form>
</div>
 ```
 ## ngModel -- two-way data binding
 ```ts
 export class DemoFormNgModelComponent implements OnInit {

  myForm:FormGroup;
  productName:string;

  constructor(fb:FormBuilder) {
    this.myForm = fb.group({
      'productName':['',Validators.required]
    });
  }

  onSubmit(value:string):void {
    console.log('you submitted value:',value);
  }

  ngOnInit() {
  }

}
 ```

 ```html
 <div class="ui raised segment">
  <h2 class="ui header">Demo Form: with ng-model</h2>

  <div class="ui info message">
    The product name is: {{productName}}
  </div>

  <form [formGroup]="myForm"
        (ngSubmit)="onSubmit(myForm.value)"
        class="ui form">

    <div class="field">
      <label for="productNameInput">Product Name</label>
      <input type="text"
             id="productNameInput"
             placeholder="Product Name"
             [formControl]="myForm.get('productName')"
             [(ngModel)]="productName">
    </div>

    <div *ngIf="!myForm.valid"
         class="ui error message">Form is invalid</div>
    <button type="submit" class="ui button">Submit</button>
  </form>

</div>
 ```