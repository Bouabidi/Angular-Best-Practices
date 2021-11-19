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
````javascript
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
```
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
````
##### const vs let


