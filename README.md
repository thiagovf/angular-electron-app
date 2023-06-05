#### Description

This project is a basic example of building an Electron app with Angular.  It can also be used to add Electron to an existing Angular app.  

Many tutorials focus on simply setting up Angular with Electron from the perspective of building an Electron app.  This tutorial will focus on a configuration that allows you to both build a web-based Angular application while retaining the ability to also build a desktop version with Electron in addition to it.


#### Stackblitz

  View [Stackblitz Example](https://stackblitz.com/github/midsever/electron-angular-tut-test).


#### Version

```
  Angular CLI : 7.1.4
  Node        : 11.6.0
  NPM         : 6.5.0
  Electron    : 4.0.0
```
  

#### 1.  Make sure you have the latest version of Angular CLI:

```shell
> npm install -g @angular/cli
```
	

#### 2.  Create a new Angular app:

```shell
> ng new my-project
```
	
  
#### 3.  Go to the new projects folder:

```shell
> cd my-project
```
	
	
#### 4.  Install latest version of Electron as a dev dependency:

```shell
> npm i --save-dev electron@latest
```
	
	
#### 5.  Create src/electron folder and src/electron/main.ts file:

	src/electron/main.ts:
	
```JSONasJs
const { app, BrowserWindow } = require('electron');
const path = require('path');
const url = require('url');

let win;

function createWindow() {
  win = new BrowserWindow({ width: 800, height: 600 });

  // load the dist folder from Angular
  win.loadURL(
  url.format({
    pathname: path.resolve(__dirname, `../my-project/index.html`), // <-- Note that we are resolving the path to the Angular apps dist location.
    protocol: 'file:',
    slashes: true
  })
  );

  // The following is optional and will open the DevTools:
  // win.webContents.openDevTools()

  win.on('closed', () => {
  win = null;
  });
}

app.on('ready', createWindow);

// on macOS, closing the window doesn't quit the app
app.on('window-all-closed', () => {
  if (process.platform !== 'darwin') {
  app.quit();
  }
});

// initialize the app's main window
app.on('activate', () => {
  if (win === null) {
  createWindow();
  }
});
```
	
Note: This is our main Electron file.  You can call it whatever you want.
	
	
#### 6.  Create the tsconfig file for electron:

Create a new file called tsconfig.electron.json in src folder: src/tsconfig.electron.json
	
	src/tsconfig.electron.json:
  
```JSONasJs
{
  "extends": "../tsconfig.json",
  "compilerOptions": {
  "outDir": "../dist/electron",    // <-- Note that we're making electron parallel to the app.
  "types": ["node"]                // <-- We need the "node" type for typescript.
  },
  "baseUrl": ".",
  "include": [
  "electron/main.ts"               // <-- This is the only file we'll be transpiling to JS.
  ]
}
```

Note: Electron depends on a JS file but since we're building with Angular and Angular uses Typescript it only makes sense to build everything with Typescript if we can.  
	
	
#### 7.  Open src/tsconfig.app.json and exclude the electron folder:

	src/tsconfig.electron.json:
```JSONasJs
{
  "extends": "../tsconfig.json",
  "compilerOptions": {
  "outDir": "../dist/app",
  "types": []
  },
  "exclude": [
  "test.ts",
  "electron",                      // <-- Note that we are expluding the electron folder.
  "**/*.spec.ts"
  ]
}
```
	
Note: We don't want anything in the electron folder transpiled into our Angular app.  Also, if we don't exclude the electron folder, the Angular "build" builder will try to transpile it and we'll get errors because we're not including the "node" type in our "types" compiler option.
	
Note: We don't need to do anything with the src/tsconfig.spec.json file because it specifies only the files for testing and ignores everything else by default.
	
	
#### 8.  Create a script to build and start electron:

	package.json:
```JSONasJs
{
  "name": "electron-angular",
  "version": "0.0.0",
  "scripts": {
  "ng": "ng",
  "start": "ng serve",
  "build": "ng build",
  "test": "ng test",
  "lint": "ng lint",
  "e2e": "ng e2e",
  "electron-tsc": "tsc -p src/tsconfig.electron.json && ng build --base-href ./ && electron ./dist/electron/main.js"
  },
  ...
}
```
	
Note: We added "electron-tsc" which does three things.  
1.  "tsc -p src/tsconfig.electron.json" transpiles our Electron Typescript file into Javascript (in the dist folder).
2.  "ng build --base-href" builds the actual Angular application and sets the base url in our index page.  
3.  "electron ./dist/electron/main.js" starts the Electron app we're specifying.

