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
