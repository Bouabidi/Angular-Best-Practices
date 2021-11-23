# Angular-Best-Practices
Angular, Best Practices, htmlhint, stylelint, prettierrc, proxy

In order to deliver a hight quality angular application , you have to follow a series of guidelines and good practices.

## Configure tsconfig.json
this file specifies the root files and the compiler options for an angular project.

## Create .eslintrc
Then add the command to script section of package.json
```javascript
"lint:ts":"eslint --color -c .eslintrc --ext .ts ."
```

## Configure HTMLHint

install htmlhint

```javascript
npm install --save-dev htmlhint
```

create a .htmlhintrc configuration file in the root of your project.

```javascript
{
"attr-value-not-empty": false
}
```

run htmlhint on, for example, all the html files in your project.

```javascript
"lint:html": "npx htmlhint \"src\"  --config .htmlhintrc ",
```

configure rules baesd on rules list of htmlhintrc

## Configure stylelint

install stylelint

```javascript
npm install --save-dev   stylelint stxlelint-config-standard
```

create .htmlhintrc  and add 

```javascript
{
"extends": "stylelint-config-standard"
}
```

run htmlhint 


```javascript
"lint:scss": "npx stylelint \"src/**/*.scss\" --syntax scss",
```

## Configure prettier

configure .prettierrc

## Configure proxy for API calls

create a file proxy.config.json  next to package.json

add the following content 
```javascript
{
  "/folder/sub-folder/*": {
    "target": "http://localhost:3000",
    "secure": false,
    "pathRewrite": {
      "^/folder/sub-folder/": "/new-folder/"
    },
    "changeOrigin": true,
    "logLevel": "debug"
  }
}
```


edit package.json file start script to be
```javascript
"start": "ng serve --proxy-config proxy.config.json"
```

relaunch the npm start

## Architectural principles
### SOLID
#### Single Responsibiliy:
A class should have only one reason to change.
help to produce more loosely coupled and modular systems.
#### Open Closed
open for extension:objects and entities able to add new features or components.

closed for modification: not introducing breaking changes to existing functionality because it can influence a lot of existing code.
#### Liskov substitution
Subclass should override the parent class method in a way that does not break functionality.
#### Interface Segregation
client should never be forced to implement an interface that it doesn't use.
#### Dependency Inversion
Entities must depend on abstraction not on concretions.

### Don't repeat yourself(DRY)
The application should avoid specifying behavior related to a particular concept in multiple places as this practice is a frequent source of errors. At some point, a change in requirements will require changing this behavior.
Rather than duplicating logic, encapsulate it in a programming construct.

### Keep it Short and Simple (KISS Principle)
design principle which states that designs and/or systems should be as simple as possible.
### Mock API with mirage.js

### Angular Features

## Directive
Should be stored in Directives folder of Shared Module.

## Pipe
### Pure Pipe
Called when Angular detects a change in the value or parameters passed.
### Impure Pipe
called for every change detection for example for mouse-move

### Angular Forms
## Basic Setup

```javascript
import {
  ChangeDetectionStrategy,
  Component,
  Input,
  OnInit
} from "@angular/core";
import { FormControl, FormGroup, Validators } from "@angular/forms";

@Component({
  selector: "lib-form",
  templateUrl: "./form.component.html",
  styleUrls: ["./form.component.scss"],
  changeDetection: ChangeDetectionStrategy.OnPush
})
export class FormGeneralComponent implements OnInit {
  public form = new FormGroup({
    name: new FormControl("", [Validators.required]),
    description: new FormControl(undefined, [Validators.required]),
    status: new FormControl(1, [Validators.required])
  });

  get controls() {
    return this.form.controls;
  }

  public ngOnInit() {
    this.initializeFormValues();
  }

  // reset form
  public reset() {
    this.form.reset();
  }

  // populate form values
  public initializeFormValues() {
    this.form.patchValue({
      name: "name",
      description: "description",
      status: 2
    });
  }

  // GET form values
  public onSubmit() {
    this.form.getRawValue();
  }
}
````

### Best practices for a clean code
#### trackBy
When using ngFor to loop over an array in templates, use it with a trackBy function which will return an unique identifier for each item.

When an array changes, Angular re-renders the whole DOM tree. But if you use trackBy, Angular will know which element has changed and will only make DOM changes for that particular element.

````javascript 
Before

<li *ngFor="let item of items;">{{ item }}</li>

After

// in the template

<li *ngFor="let item of items; trackBy: trackByFn">{{ item }}</li>

// in the component

trackByFn(index, item) {    
   return item.id; // unique id corresponding to the item
}
```
##### const vs let
```javascript
Before

let car = 'ludicrous car';

let myCar = `My ${car}`;
let yourCar = `Your ${car};

if (iHaveMoreThanOneCar) {
   myCar = `${myCar}s`;
}

if (youHaveMoreThanOneCar) {
   yourCar = `${youCar}s`;
}
After

// the value of car is not reassigned, so we can make it a const
const car = 'ludicrous car';

let myCar = `My ${car}`;
let yourCar = `Your ${car};

if (iHaveMoreThanOneCar) {
   myCar = `${myCar}s`;
}

if (youHaveMoreThanOneCar) {
   yourCar = `${youCar}s`;
}


````
#### pipeable operators
Pipeable operators are tree-shakeable meaning only the code we need to execute will be included when they are imported.

This also makes it easy to identify unused operators in the files.

```javascript
Before

import 'rxjs/add/operator/map';
import 'rxjs/add/operator/take';

iAmAnObservable
    .map(value => value.item)
    .take(1);
After

import { map, take } from 'rxjs/operators';

iAmAnObservable
    .pipe(
       map(value => value.item),
       take(1)
     );
```
##### Subscribe in template
Avoid subscribing to observables from components and instead subscribe to the observables from the template.
```javascript
Before

// // template

<p>{{ textToDisplay }}</p>

// component

iAmAnObservable
    .pipe(
       map(value => value.item),
       takeUntil(this._destroyed$)
     )
    .subscribe(item => this.textToDisplay = item);
After

// template

<p>{{ textToDisplay$ | async }}</p>

// component

this.textToDisplay$ = iAmAnObservable
    .pipe(
       map(value => value.item)
     );
```

#### Clean up subscriptions
Failing to unsubscribe from observables will lead to unwanted memory leaks as the observable stream is left open, potentially even after a component has been destroyed / the user has navigated to another page.

````javascript

Before

iAmAnObservable
    .pipe(
       map(value => value.item)     
     )
    .subscribe(item => this.textToDisplay = item);
After

Using takeUntil when you want to listen to the changes until another observable emits a value:

private _destroyed$ = new Subject();

public ngOnInit (): void {
    iAmAnObservable
    .pipe(
       map(value => value.item)
      // We want to listen to iAmAnObservable until the component is destroyed,
       takeUntil(this._destroyed$)
     )
    .subscribe(item => this.textToDisplay = item);
}

public ngOnDestroy (): void {
    this._destroyed$.next();
    this._destroyed$.complete();
}
````

#### Lazy load
When possible, try to lazy load the modules in your Angular application. Lazy loading is when you load something only when it is used.
This will reduce the size of the application to be loaded and can improve the application boot time by not loading the modules that are not used.
```javascript
Before

// app.routing.ts

{ path: 'not-lazy-loaded', component: NotLazyLoadedComponent }
After

// app.routing.ts

{ 
  path: 'lazy-load',
  loadChildren: 'lazy-load.module#LazyLoadModule' 
}

// lazy-load.module.ts
import { NgModule } from '@angular/core';
import { CommonModule } from '@angular/common';
import { RouterModule } from '@angular/router';
import { LazyLoadComponent }   from './lazy-load.component';

@NgModule({
  imports: [
    CommonModule,
    RouterModule.forChild([
         { 
             path: '',
             component: LazyLoadComponent 
         }
    ])
  ],
  declarations: [
    LazyLoadComponent
  ]
})
export class LazyModule {}
```

#### Avoid subscriptions inside subscriptions
Avoid subscribing to one observable in the subscribe block of another observable. Instead, use appropriate chaining operators. Chaining operators run on observables from the operator before them. Some chaining operators are: withLatestFrom, combineLatest, etc.

```javascript
Before

firstObservable$.pipe(
   take(1)
)
.subscribe(firstValue => {
    secondObservable$.pipe(
        take(1)
    )
    .subscribe(secondValue => {
        console.log(`Combined values are: ${firstValue} & ${secondValue}`);
    });
});
After

firstObservable$.pipe(
    withLatestFrom(secondObservable$),
    first()
)
.subscribe(([firstValue, secondValue]) => {
    console.log(`Combined values are: ${firstValue} & ${secondValue}`);
});
```

Why?

Code smell/readability/complexity: Not using RxJs to its full extent, suggests developer is not familiar with the RxJs API surface area.

Performance: If the observables are cold, it will subscribe to firstObservable, wait for it to complete, THEN start the second observable’s work. If these were network requests it would show as synchronous/waterfall.

#### Avoid any; type everything
Always declare variables or constants with a type other than any.

#### Small reusable components

Extract the pieces that can be reused in a component and make it a new one. Make the component as dumb as possible, as this will make it work in more scenarios. Making a component dumb means that the component does not have any special logic in it and operates purely based on the inputs and outputs provided to it.

Reusable components reduce duplication of code therefore making it easier to maintain and make changes.

#### Components should only deal with display logic

Avoid having any logic other than display logic in your component whenever you can and make the component deal only with the display logic.

Components are designed for presentational purposes and control what the view should do. Any business logic should be extracted into its own methods/services where appropriate, separating business logic from view logic.

Business logic is usually easier to unit test when extracted out to a service, and can be reused by any other components that need the same business logic applied.

#### Avoid long methods

Long methods generally indicate that they are doing too many things. Try to use the Single Responsibility Principle. 

Long methods are hard to read, understand and maintain. They are also prone to bugs, as changing one thing might affect a lot of other things in that method. 

#### Add caching mechanisms
When making API calls, responses from some of them do not change often. In those cases, you can add a caching mechanism and store the value from the API. When another request to the same API is made, check if there is a value for it in the cache and if so, use it. Otherwise, make the API call and cache the result.

Having a caching mechanism means avoiding unwanted API calls. By only making the API calls when required and avoiding duplication, the speed of the application improves as we do not have to wait for the network. It also means we do not download the same information over and over again.

#### Avoid logic in templates
If you have any sort of logic in your templates, even if it is a simple && clause, it is good to extract it out into its component.

Having logic in the template means that it is not possible to unit test it and therefore it is more prone to bugs when changing template code.

```javascript
Before

// template
<p *ngIf="role==='developer'"> Status: Developer </p>

// component
public ngOnInit (): void {
    this.role = 'developer';
}
After

// template
<p *ngIf="showDeveloperStatus"> Status: Developer </p>

// component
public ngOnInit (): void {
    this.role = 'developer';
    this.showDeveloperStatus = true;
}

```

#### Strings should be safe
If you have a variable of type string that can have only a set of values, instead of declaring it as a string type, you can declare the list of possible values as the type.

By declaring the type of the variable appropriately, we can avoid bugs while writing the code during compile time rather than during runtime.

```javascript
Before

private myStringValue: string;

if (itShouldHaveFirstValue) {
   myStringValue = 'First';
} else {
   myStringValue = 'Second'
}
After

private myStringValue: 'First' | 'Second';

if (itShouldHaveFirstValue) {
   myStringValue = 'First';
} else {
   myStringValue = 'Other'
}

// This will give the below error
Type '"Other"' is not assignable to type '"First" | "Second"'
(property) AppComponent.myValue: "First" | "Second" 
```

#### Write Small pure functions
This avoids functions like this:
```javascript
addOrUpdateData(data: Data, status: boolean) {
  if (status) {
    return this.http.post<Data>(url, data)
      .pipe(this.catchHttpErrors());
  }
  return this.http.put<Data>(`${url}/${data.id}`, data)
    .pipe(this.catchHttpErrors());
  }
}

```

And hopefully more like this:

```javascript
addData(data: Data) {
  return this.http.post<Data>(url, data)
    .pipe(this.catchHttpErrors());
}
updateData(data: Data) {
  return this.http.put<Data>(`${url}/${data.id}`, data)
    .pipe(this.catchHttpErrors());
}

```

#### Remove unused code
It is extremely valuable to know if a piece of code is being used or not. If you let unused code stay, then in the future, it can be hard or almost impossible to be certain if it’s actually used or not.


#### Separation of Concerns

You should create something like a “common frame” for your application where you will include common components. This comes in handy when you don’t want to contaminate your components with the same code. Since Angular doesn’t allow us to import component directly between different modules, we need to put these components inside the shared module.

```javascript
// src/app/shared/components/reusable/resuable.component
export class ReusableComponent implements OnInit {
  @Input() title: string;
  @Input() body: string;
 
  @Output() buttonClick = new EventEmitter();
  constructor() { }
  ngOnInit() {}
  onButtonClick(){
    this.buttonClick.emit('Button was clicked');
  }
}
--------------------------------------------------------------------
// You can then use this component directly inside the components of
// your choice
// src/app/some/some.component
@Component({
  selector: 'app-some',
  template: `<app-reusable [title]="'Awesome title!'"
               [body]="'Lorem ipsum'"
               (buttonClick)="buttonClick($event)>
             </app-reusable>`,
})
export class SomeComponent implements OnInit {
  constructor() {}
  ngOnInit() {}
  public buttonClick(e){
    console.log(e);
  }
}
```

Quick tip! We’re able to control the HTML contents from the “parent” component by using the ng-content tag. Read more about content projection with ng-content.

#### Using Interfaces
If we want to create a contract for our class we should always use interfaces. By using them we can force the class to implement functions and properties declared inside the interface.
Using interfaces is a perfect way of describing our object literals. If our object is of an interface type, it is obligated to implement all of the interface’s properties. We shouldn’t name our interfaces with the starting capital I letter as we do in some programming languages.
````javascript
export interface User {
    name: string;
    age: number;
    address: string;
} 
````

We can specify optional properties, by using the question mark (?) inside an interface as well. We don’t need to populate those properties inside an object:

#### Using Immutability
Objects and arrays are the reference types in javascript. If we want to copy them into another object or an array and to modify them, the best practice is to do that in an immutable way.
By modifying reference types immutably, we are preserving the original objects and arrays and modifying only their copies.
The easiest way to modify objects and arrays immutably is by using the es6 spread operator (…) :
````javascript
this.user = {
  name: 'Dzon',
  age: 25,
  address: 'Sunny street 34'
}
let updatedUser = {
  ...this.user,
  name: 'Peter'
}
````
We are deep copying the user object and then just overriding the name property.

````javascript
public results = [10, 12, 14];
let newNumbers = [...this.numbers, 45, 56];
````

#### Safe Navigation Operator (?) in HTML Template

To be on the safe side we should always use the safe navigation operator while accessing a property from an object in a component’s template. If the object is null and we try to access a property, we are going to get an exception. But if we use the save navigation (?) operator, the template will ignore the null value and will access the property once the object is not the null anymore.

```javascript
<div class="col-md-3">
  {{user?.name}}
</div>
```

#### Components, Decorators, Directives, and Lifecycle

* Reusable Components and Decorators
Creating reusable components is one of the best techniques we can use while developing our project. We can reuse those types of components inside any parent component and pass the data through the @Input decorator. Those components can emit events by using the @Output decorator and EventEmmiter class. Here is an example of the @Input and @Output decorators in action:

```javascript 
export class SuccessModalComponent implements OnInit {
  @Input() public modalHeaderText: string;
  @Input() public modalBodyText: string;
  @Input() public okButtonText: string;
  @Output() public redirectOnOK = new EventEmitter();
  constructor() { }
  ngOnInit() {
  }
  public emmitEvent(){
    this.redirectOnOK.emit();
  }
```

* Using Directives
Whenever we have a situation where multiple HTML elements have the same behavior (for example: when we hover over the element it receives the blue color), we should consider using attribute directives. We shouldn’t repeat the hover logic every time we need it on some HTML element. A much better way would be to create a directive and then just reuse it on the particular element.

```javascript
import { Directive, ElementRef, HostListener, Input } from '@angular/core';
@Directive({
  selector: '[appHover]'
})
export class HoverDirective {
  @Input() public hoverColor: string;
  constructor(private element: ElementRef){}
  @HostListener('mouseenter') onMouseEnter(){
    this.highlightElement(this.hoverColor);
  }
  @HostListener('mouseleave') onMouseLeave(){
    this.highlightElement(null);
  }
  private highlightElement(color: string){
    this.element.nativeElement.style.backgroundColor = color;
  }
}
```
We need to register this directive in a required module and to call it in HTML template file:

```javascript
<p appHover [hoverColor]="'blue'">this is hoverable text</p>
```

#### Using Lifecycle Hooks
Lifecycle hooks play a very important part of Angular development. We should use them whenever we have an opportunity to. For example, if we need to fetch some data from a database as soon as our component is instantiated, we should use ngOnInit() lifecycle hook and not the constructor.

If we have some logic inside child components and we want to execute that logic as soon as decorator parameters are modified, we can use ngOnChanges() lifecycle hook.

If we need to clean up some resources as soon as our component is destroyed, we should use ngOnDestroy() lifecycle hook.

Even though we don’t have to implement interfaces to use the lifecycle hooks, we are better off implementing them.
