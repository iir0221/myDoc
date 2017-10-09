# Build in Directives
* directive
    * attributes we add to our HTML elements that give us dynamic behavior.

## NgIf
```html
<div *ngIf="false"></div> <!--never displayed-->
<div *ngIf="a > b"></div> <!--displayed if a is more than b-->
<div *ngIf="str == 'yes'"></div> <!-- displayed if str is the string "yes" --> 
<div *ngIf="myFunc()"></div> <!-- displayed if myFunc returns truthy -->
```
## NgSwitch
```html
<div class="container" [ngSwitch]="myVar">
    <div *ngSwitchCase="'A'">Var is A</div>
    <div *ngSwitchCase="'B'">Var is B</div>
    <div *ngSwitchCase="'C'">Var is C</div>
    <div *ngSwitchDefault>Var is something else</div>
</div>
```
* ngSwitchDefault if optional

```html
<ul [ngSwitch]="choice">
<li *ngSwitchCase="1">First choice</li>
<li *ngSwitchCase="2">Second choice</li>
<li *ngSwitchCase="3">Third choice</li>
<li *ngSwitchCase="4">Fourth choice</li>
<li *ngSwitchCase="2">Second choice, again</li> <li *ngSwitchDefault>Default choice</li>
</ul>
```
in this case,if choice is 2,both second and fifth li will be rendered.
## NgStyle 
* [style.<cssproperty>]="value"
    ```html
    <div [style.background-color]="'yellow'"> 
        Uses fixed yellow background
    </div>
    ```
* [ngStyle]="{key:value,key:value...}"
    ```html
    <div [ngStyle]="{color: 'white', 'background-color': 'blue'}"> 
        Uses fixed white text on blue background
    </div>
    ```
    * 因为'-'不是一个合法的key字符，所以'background-color'用''包起来
* 动态指定css style
    ```html
    <div class="ui input">
        <input type="text" name="color" value="{{color}}" #colorinput>
    </div>
    <div class="ui input">
        <input type="text" name="fontSize" value="{{fontSize}}" #fontinput>
    </div>
    <button class="ui primary button" (click)="apply(colorinput.value, fontinput.val\ ue)">
        Apply settings 
    </button>
    ```
    ```html
    <div>
        <span [ngStyle]="{color: 'red'}" [style.font-size.px]="fontSize">
            red text 
        </span>
    </div>

    <h4 class="ui horizontal divider header"> 
        ngStyle with object property from variable
    </h4>
    <div>
        <span [ngStyle]="{color: color}">
            {{ color }} text 
        </span>
    </div>
    <h4 class="ui horizontal divider header"> 
        style from variable
    </h4>
    <div [style.background-color]="color" style="color: white;">
        {{ color }} background 
    </div>
    ```
## NgClass
dynamic CSS classes
* [ngClass]="{CssClassName:true/false,CssClassName:true/false,...}"
```css
.bordered {
border: 1px dashed black; background-color: #eee; }
```
```html
<div [ngClass]="{bordered: false}">This is never bordered</div> 
<div [ngClass]="{bordered: true}">This is always bordered</div>
```
* dynamic
```html
<div [ngClass]="{bordered: isBordered}">
    Using object literal. Border {{ isBordered ? "ON" : "OFF" }}
</div>
```


## NgFor
* *ngFor="let item of items"
* Getting an index
    * 
    ```html
    <div class="ui list" *ngFor="let c of cities; let num = index"> 
        <div class="item">{{ num+1 }} - {{ c }}</div>
    </div>
    ```

## NgNonBindable
```html
<div class='ngNonBindableDemo'>
    <span class="bordered">{{ content }}</span> 
    <span class="pre" ngNonBindable>
        &larr; This is what {{ content }} rendered 
    </span>
</div>
```
* {{ content }}将不会被解析而直接render

