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

```javascript 
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
```
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

#### Communicating across Components: reusable components
how to create reusable components in Angular. I’ll be making use of Angular’s @Input and @Output directives?

##### @Input
@Input decorator allows parent component to bind it's properties to child component and thus gives the child component access to its data. These bindings are actually a reference to properties on the parent component.

Say we have two components CourseListComponent to the CourseItemComponent and we want to pass "course" information from our CourseListComponent to the CourseItemComponent.

we have our course model
```javascript
export class Course {
  public name: string;
  public description: string;
  public courseImagePath: string;

  constructor(name: string, desc: string, courseImagePath: string) {
    this.name = name;
    this.description = desc;
    this.courseImagePath = imagePath;
  }
}
```
our course-list component looks something like this:
```javascript
...
export class CourseListComponent implements OnInit {
  courses: Course[] = [
    new Course('Angular Course 1', 'This is simply a practice Angular course 1', 'http://via.digital.com/350x150.jpeg'),
    new Course('Angular Course 2', 'This is simply a practice Angular course 2', 'http://via.digital.com/350x150'),
	new Course('Angular Course 3', 'This is simply a practice Angular course 3', 'http://via.digital.com/350x150')
  ];

  constructor() { }

  ngOnInit() {
  }
}
```
Below is how we would update our course-list template:
```javascript
<div class="row">
  <div class="col-xs-12">
    <app-course-item
      *ngFor="let courseEl of courses"
      [course]="courseEl"></app-course-item>
  </div>
</div>
```
We pass the individual course element for that iteration to the CourseItemComponent. Thus, we are binding the "course" property of the child CourseItemComponent from the parent CourseListComponent.

Now, to be able to do this, we have to use the @Input decorator on the child CourseItemComponent. Thus, our updated CourseItemComponent would look like below:
```javascript
export class CourseItemComponent implements OnInit {
@Input() course: Course;  
  constructor() { }
  ngOnInit() {  }
}
```
We can now render the course name/description/image in the CourseItemComponent template as below:
```javascript
<a
  href="#"
  class="list-group-item clearfix">
  <div class="pull-left">
    <h4 class="list-group-item-heading">{{ course.name }}</h4>
    <p class="list-group-item-text">{{ course.description }}</p>
  </div>
  <span class="pull-right">
        <img
          [src]="course.courseImagePath"
          alt="{{ course.name }}"
          class="img-responsive"
          style="max-height: 50px;">
      </span>
</a> 
```
##### Using @Output Decorator and EventEmitter (Event Binding)
In certain cases, we want to be able to emit data back from the child component to the parent component. There is an EventEmitter property exposed by the Child component exposes, which would emit data whenever any action/event occurs on the child component.

So, we'll have a "click" event on the CourseItemComponent. This is how the template for that would be updated to:

```javascript
<a
  href="#"
  class="list-group-item clearfix"
	(click)="onSelected()">
  <div class="pull-left">
    <h4 class="list-group-item-heading">{{ course.name }}</h4>
    <p class="list-group-item-text">{{ course.description }}</p>
  </div>
  <span class="pull-right">
        <img
          [src]="course.courseImagePath"
          alt="{{ course.name }}"
          class="img-responsive"
          style="max-height: 50px;">
      </span>
</a>
```
Now, from our CourseItemComponent typescript code, we want to be able to emit this event and inform the parent component about the event. If we want "courseSelected" to be listenable from outside, we have to use the @Output decorator. This would give access to emit() method on this property. 

```javascript
export class CourseItemComponent implements OnInit {
@Input() course: Course;  
@Output() courseSelected = new EventEmitter<Course>();
  constructor() { }
  ngOnInit() {  }
onSelected() {
    this.courseSelected.emit(this.course);
  }
}
```
We would now be able to listen to the "courseSelected" event on the parent CourseListComponent as shown below.

```javascript
<div class="row">
  <div class="col-xs-12">
    <button class="btn btn-success">New Course</button>
  </div>
</div>
<hr>
<div class="row">
  <div class="col-xs-12">
    <app-course-item
      *ngFor="let courseEl of courses"
      [course]="courseEl"
      (courseSelected)="onCourseSelected(courseEl)"></app-course-item>
  </div>
</div>
```
Also, we'll now get access to the "course" in our CourseListComponent typescript code.

```javascript
export class CourseListComponent implements OnInit {
  courses: Course[] = [
    new Course('Angular Course 1', 'This is simply a practice Angular course 1', 'http://via.digital.com/350x150.jpeg'),
    new Course('Angular Course 2', 'This is simply a practice Angular course 2', 'http://via.digital.com/350x150')
  ];

  constructor() { }

  ngOnInit() {
  }

onCourseSelected(course: Course) {
   // console.log("Selected course : " + course);
  }
}
```

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

#### Path Alias
Path aliases simplify paths by giving a link to the path rather than using the the full relative path.
```javascript
// before:
import { SharedModule } from '../../shared/shared.module';
import { DatePipe } from '../../../date.pipe';
import { environment } from '../../../environments/environment';
 
// after:
import { SharedModule } from '@shared/shared.module';
import { DatePipe } from '@shared/pipes/date.pipe';
import { environment } from '@env/environment';
```
We add some change to tsconfig.json
```javascript
{
  "compilerOptions": {
    ...,  
    "baseUrl": "./",
    "paths": {
      "@app/*": [
        "src/app/*"
      ],
      "@core/*": [
        "src/app/core/*"
      ],
      "@shared/*": [
        "src/app/shared/*"
      ],
      "@env/*": [
        "src/environments/*"
      ]
    }
  }
}
```
#### Consume REST API in Angular
Most applications need to communicate with a remote server over the HTTP protocol, in order to perform the basic CRUD operations. With Angular, you can use HTTPClient service to achieve this communication easily. As an example, if you need to manage the Posts of your blog, you may have the following service to handle all the operations on the Post resource:
```javascript
import { Injectable } from '@angular/core';
import { HttpClient, HttpErrorResponse } from '@angular/common/http';
import { environment } from '../../environments/environment';
import { Observable, throwError } from 'rxjs';
import { catchError } from 'rxjs/operators';

@Injectable({
  providedIn: 'root'
})
export class PostService {
  private readonly APIUrl = environment.APIUrl + 'posts';

  constructor(private httpClient: HttpClient ) { }

  getList(page: number, count: number): Observable<Post[]> {
    let params = new HttpParams()
			.set('page', page.toString())
			.set('count', count.toString());

    return this.httpClient.get<Post[]>(`/${this.APIUrl}}?${params.toString()}`)
      .pipe(
        catchError(this.handleError)
      );
  }

  get(id: string | number): Observable<Post> {
    return this.httpClient.get<Post>(`/${this.APIUrl}}/${id}`)
      .pipe(
        catchError(this.handleError)
      );
  }
  
  add(post: Post): Observable<any> {
    return this.httpClient.post(`/${this.APIUrl}}`, post)
      .pipe(
        catchError(this.handleError)
      );
  }

  delete(id: string | number): Observable<any> {
    return this.httpClient.delete(`/${this.APIUrl}}/${id}`) 
      .pipe(
        catchError(this.handleError)
      );
  }

  update(post: Post) {
    return this.httpClient.put(`/${this.APIUrl}}`,post)
      .pipe(
        catchError(this.handleError)
      );
  }
  
  private handleError(error: HttpErrorResponse) {
    // Handle the HTTP error here
    return throwError('Something wrong happened');
  }
}
```
This solution is simple and clean, and it even follows the best practices according to the official Angular Documentation. However, applications usually have many resources to manage, for our example, we may have users, comments, reviews, etc. Ideally, each of these resources should have a separate service to handle CRUD operations and communicate with the server, at the end we will have UserService, CommentService, ReviewService.Let’s take a look at how the
```javascript
import { Injectable } from '@angular/core';
import { HttpClient, HttpErrorResponse } from '@angular/common/http';
import { environment } from '../../environments/environment';
import { Observable, throwError } from 'rxjs';
import { catchError } from 'rxjs/operators';

@Injectable({
  providedIn: 'root'
})
export class CommentService {
  private readonly APIUrl = environment.APIUrl + 'comments';

  constructor(private httpClient: HttpClient ) { }

  getList(index: number, page: number): Observable<Comment[]> {
    let params = new HttpParams()
			.set('limit', index.toString())
			.set('offset', page.toString());

    return this.httpClient.get<Comment[]>(`/${this.APIUrl}}?${params.toString()}`)
      .pipe(
        catchError(this.handleError)
      );
  }

  get(id: string | number): Observable<Comment> {
    return this.httpClient.get<Comment>(`/${this.APIUrl}}/${id}`)
      .pipe(
        catchError(this.handleError)
      );
  }
  
  add(comment: Comment): Observable<any> {
    return this.httpClient.post(`/${this.APIUrl}}`, comment)
      .pipe(
        catchError(this.handleError)
      );
  }

  delete(id: string | number): Observable<any> {
    return this.httpClient.delete(`/${this.APIUrl}}/${id}`) 
      .pipe(
        catchError(this.handleError)
      );
  }

  update(comment: Comment) {
    return this.httpClient.put(`/${this.APIUrl}}`,comment)
      .pipe(
        catchError(this.handleError)
      );
  }
  
  private handleError(error: HttpErrorResponse) {
    // Handle the HTTP error here
    return throwError('Something wrong happened');
  }
}
```
* The Problem
Although the above implementation is very common and widly acceptable, it sufferes from two cons:
1. Code redundant (breaking of the DRY principle): If you compare the PostService and the CommentService you will notice how redundant the code is.
2. Changes in the server-side, or changes in the way to communicate to the server, require changes in many files (in our case we need to change both PostService and CommentService files)

* Typescript Generics To The Rescue
To solve the above issues let’s go ahead and build the following abstract class which will be the base of all the other services:
```javascript
import { Injectable } from '@angular/core';
import { HttpClient, HttpErrorResponse, HttpParams } from '@angular/common/http';
import { environment } from '../../environments/environment';
import { Observable, throwError } from 'rxjs';
import { catchError } from 'rxjs/operators';

@Injectable({
  providedIn: 'root'
})
export abstract class ResourceService<T> {
  private readonly APIUrl = environment.APIUrl + this.getResourceUrl();

  constructor(protected httpClient: HttpClient ) {
  }

  abstract getResourceUrl(): string;

   getList(page: number, count: number): Observable<T[]> {
    let params = new HttpParams()
			.set('page', page.toString())
			.set('count', count.toString());

    return this.httpClient.get<T[]>(`/${this.APIUrl}}?${params.toString()}`)
      .pipe(
        catchError(this.handleError)
      );
  }

  get(id: string | number): Observable<T> {
    return this.httpClient.get<T>(`/${this.APIUrl}}/${id}`)
      .pipe(
        catchError(this.handleError)
      );
  }
  
  add(resource: T): Observable<any> {
    return this.httpClient.post(`/${this.APIUrl}}`, resource)
      .pipe(
        catchError(this.handleError)
      );
  }

  delete(id: string | number): Observable<any> {
    return this.httpClient.delete(`/${this.APIUrl}}/${id}`) 
      .pipe(
        catchError(this.handleError)
      );
  }

  update(resource: T) {
    return this.httpClient.put(`/${this.APIUrl}}`,resource)
      .pipe(
        catchError(this.handleError)
      );
  }
  
  private handleError(error: HttpErrorResponse) {
    // Handle the HTTP error here
    return throwError('Something wrong happened');
  }
}
```
1. The new service class is abstract, which means it can’t be instantiated and used directly, but it needs to be extended by other classes.
2. We provide one abstract method getResourceUrl, The class which extends this abstract class must implement this method, and return the URL of the resource as we will see in the following section.
3. This is a Generic Class, it is not tied to a specific type, rather the class which extends this abstract class will define the exact type used.
4. It has all the needed CRUD operations which we need and used before in the previous service.

Now after we have our abstract generic class, whenever we need a new service we can simply extend this class and implement the only one abstract method getResourceUrl.

* Server vs Front-end Model

In most applications, the front-end model doesn’t match %100 the server-side model. In other words, the REST API will respond with json object that doesn’t match exactly the interface or the class defined in the front-end application. In this case, you need a mapping function to convert between server and front-side mode. This sometimes referred to as serializing/deserializing.

So, let us extend our base class to provide this mapping functionality. To do so I updated the ResourceService to look as the following:
```javascript
import { Injectable } from '@angular/core';
import { HttpClient, HttpErrorResponse, HttpParams } from '@angular/common/http';
import { environment } from '../../environments/environment';
import { Observable, throwError } from 'rxjs';
import { catchError, map } from 'rxjs/operators';

@Injectable({
  providedIn: 'root'
})
export abstract class ResourceService<T> {
  private readonly APIUrl = environment.APIUrl + this.getResourceUrl();

  constructor(protected httpClient: HttpClient ) {
  }

  abstract getResourceUrl(): string;

  toServerModel(entity: T): any {
    return entity;
  }

  fromServerModel(json: any): T {
    return json;
  }

  getList(index: number, page: number): Observable<T[]> {
    let params = new HttpParams()
			.set('limit', index.toString())
			.set('offset', page.toString());

    return this.httpClient.get<T[]>(`/${this.APIUrl}?${params.toString()}`)
      .pipe(
        map((list) => list.map((item)=> this.fromServerModel(item))),
        catchError(this.handleError)
      );
  }

  get(id: string | number): Observable<T> {
    return this.httpClient.get<T>(`/${this.APIUrl}/${id}`)
      .pipe(
        map((json) => this.fromServerModel(json)),
        catchError(this.handleError)
      );
  }
  
  add(resource: T): Observable<any> {
    return this.httpClient.post(`/${this.APIUrl}`, this.toServerModel(resource))
      .pipe(
        catchError(this.handleError)
      );
  }

  delete(id: string | number): Observable<any> {
    return this.httpClient.delete(`/${this.APIUrl}/${id}`) 
      .pipe(
        catchError(this.handleError)
      );
  }

  update(resource: T) {
    return this.httpClient.put(`/${this.APIUrl}`, this.toServerModel(resource))
      .pipe(
        catchError(this.handleError)
      );
  }
  
  private handleError(error: HttpErrorResponse) {
    // Handle the HTTP error here
    return throwError('Something wrong happened');
  }
}

```
Now to consume the new changes we have two cases:
1. There is no mapping needed between the server and the front-end model, in this case, we don’t have to change anything in the class that extends the resourceService.
2. There is some kind of mapping needed between the server and the front-end model, all we need to do is to override toServerModel and fromServerModel models in the derived class to address our requirement mappings. For example let’s assume, that the PostsService implemented previously needs to map from timestamp to a js Date object, the PostsService implementation would look like the following:

```javascript
import { HttpClient } from '@angular/common/http';
import { Injectable } from '@angular/core';
import { ResourceService } from '../resource/resource.service';

@Injectable({
  providedIn: 'root'
})
export class PostsService extends ResourceService<Post>{
  
  constructor(protected httpClient: HttpClient ) { 
    super(httpClient);
  }

  getResourceUrl(): string {
    return 'posts';
  }

  toServerModel(entity: Post): any {
    return {
      ...entity,
      createDate: entity.createDate.getTime()
    };
  }

  fromServerModel(json: any): Post{
    return {
      ...json,
      createDate: new Date(json.createDate)
    };
  } 
}
```
## Containerizing Angular using Docker
### Creating Docker file
Then create a new file called Dockerfile that will be located in the project’s root folder. It should have these following lines:
```javascript
### STAGE 1: Build ###
FROM node:12.20-alpine3.10 AS build
WORKDIR /app
COPY package.json package-lock.json ./
RUN npm install
COPY . .
RUN npm run build:prod

### STAGE 2: Run ###
FROM nginx:1.17.1-alpine
COPY --from=build /app/dist/ngx-levi9 /usr/share/nginx/html
```
#### STAGE 1: Build
```javascript
FROM node:12.20-alpine3.10 As build
```
This line will tell the docker to pull the node image with tag 12.20-alpine3.10 if the images don't exist. We are also giving a friendly name build to this image so we can refer it later.
```javascript
WORKDIR /app
```
This WORKDIR command will create the working directory in our docker image. going forward any command will be run in the context of this directory.
```javascript
COPY package.json package-lock.json ./
```
This COPY command will copy package.json and package-lock.json from our current directory to the root of our working directory inside a container which is /app.
```javascript
RUN npm install
```
This RUN command will restore node_modules define in our package.json

```javascript
COPY . .
```
This COPY command copies all the files from our current directory to the container working directory. this will copy all our source files.
```javascript
RUN npm run build:prod
```
This command will build our angular project in production mode and create production ready files in dist/appName folder
#### STAGE 2: Run
```javascript
FROM nginx:1.17.1-alpine
```
This line will create a second stage nginx container where we will copy the compiled output from our build stage
```
COPY --from=build /app/dist/ngx-levi9 /usr/share/nginx/html
```
This is the final command of our docker file. This will copy the compiled angular app from builder stage path /app/dist/ngx-levi9 to nginx container

### Creating docker ignore file
When COPY . . command execute it will copy all the files in the host directory to the container working directory. if we want to ignore some folder like .git or node_modules we can define these folders in .dockerignore file.
```
.git
node_modules
```
### Building a docker image
Navigate the project folder in command prompt and run the below command to build the image

```javascript
docker build . -t dockerngstart .
```
This command will look for a docker file in the current directory and create the image with tag dockerngstart. with -t command you can specify the tag for the image the default convention is DockerHubUsername/ImageName.

### Running a container
You can run the docker image using the below command
```javascript
docker run --name dockerngstart-container -it -p 8000:80 dockerngstart
```
A container may be running, but you may not be able to interact with it. To start the container in interactive mode, use the –i and –t options

Navigate to your browser with http://localhost:8000

### Review docker images
Run docker images command to list all the docker images in your machine

```javascript
docker images
```
Remove docker image
```javascript
docker rmi <IMAGE ID>
```
### List Docker Containers
To list all running Docker containers, enter the following into a terminal window
```javascript
docker ps
```
To list all containers, both running and stopped, add –a
```javascript
docker ps -a
```

### Start Docker Container
The main command to launch or start a single or multiple stopped Docker containers is docker start:
```javascript
docker start [options] container_id 
```
To create a new container from an image and start it, use docker run
```javascript
docker run [options] image [command] [argument] 
```
### Stop Docker Container
By default, you get a 10 second grace period. The stop command instructs the container to stop services after that period. Use the --time option to define a different grace period expressed in seconds

```javascript
docker stop [option] container_id

docker stop --time=20 container_id
```
To immediately kill a docker container without waiting for the grace period to end use:
```javascript
docker kill [option] container_id
```
To stop all running containers, enter the following:
```javascript
docker stop $(docker ps –a –q)
```
## Difference between ng-template, ng-container and ng-content
### <ng-template></ng-template>
These template elements only work in the presence of structural directives, which help us to define a template that doesn’t render anything by itself, but conditionally renders them to the DOM. It helps us create dynamic templates that can be customized and configured.
```javascript
<div> 
   Ng-template Content 
   <div *ngIf=”false else showNgTemplateContent”> 
      Shouldn't be displayed 
   </div>
</div>
 
<ng-template #showNgTemplateContent> Should be displayed
</ng-template>
```
### <ng-container>

ng-container is an extremely simple directive that allows you to group elements in a template that doesn’t interfere with styles or layout because Angular doesn’t put it in the DOM
This is helpful if you don’t want any extra div on DOM, you can simply use
ng-container. For eg: If there are two structural directives are being called on one div as below:
```javascript
<div *ngIf="details" *ngFor="let info of details">
  {{ info.content }}
</div>
Attempting to compile this code will result in the following error:
Can't have multiple template bindings on one element. Use only one attribute prefixed with *
```
One workaround would be to separate the bindings as below:
```javascript
<div *ngIf="details">
  <div *ngFor="let info of details">
    {{ info.content }}
  </div>
</div>
```
Or we can use <ng-container> without adding any extra element to the DOM at runtime:
```javascript
<ng-container *ngIf="details">
  <div *ngFor="let info of details">
    {{ info.content }}
  </div>
</ng-container>	
```
### <ng-content></ng-content>
