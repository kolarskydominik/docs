# Create and Deploy Micro-Frontend App to use in Host App (React+TS+Vite)

sources:
- https://dev.to/giveitatry/microfrontends-in-reactjs-do5
- https://medium.com/@vinaykrmittal/micro-frontend-architecture-with-react-e6379b13d7d7

template:
- https://github.com/dominikkolarsky/calendar-micro-app
  

## 1. Setup a new Vite project

```shell
npm create vite@latest
? Project name: › my-micro-app
? Select a framework: › React
? Select a variant: › TypeScript + SWC
cd my-micro-app
npm i
```

#### 1.1 Early first commit

```shell
git init
git add --all
git commit -m "blank vite react/typescript project"
```
_This way you can easily spot what you have done and what has been created by create vite@latest for example._

#### 1.2 Create a .nvmrc file

```shell
node --version > .nvmrc
git add .nvmrc
git commit -m "created .nvmrc file"
```
_Having done that, any developer can just run **nvm use** in the project folder and nvm will automatically switch to the correct version of node._

#### (1.3. Install node's types)

```shell
npm install @types/node -D
```


## 2. Setup/Cleanup App.tsx, main.tsx, index.html

### 2.1 main.tsx

```js
// main.tsx

import React from 'react';
import ReactDOM from 'react-dom/client';
import App from './App.tsx';
import './index.css';

const widgetName = 'name-of-micro-app';
const widgetDiv = document.getElementById(widgetName);

ReactDOM.createRoot(widgetDiv!).render(
  <React.StrictMode>
    <App domElement={widgetDiv!} />
  </React.StrictMode>
);

```

### 2.2 App.tsx
This is the code, which will be injected in Host App.

```js
// App.tsx

function App({ domElement }: { domElement: HTMLElement }) {
  const attribute = domElement.getAttribute('data-attribute-1');

  return (
    <>
      <div>Hello from micro-frontend-app, called: {attribute}</div>
    </>
  );
}

export default App;
```
_DomElement parameter used to read the configuration of the micro-frontend-app. The configuration will be filled in Host App and passed as data attributes to our micro-app._


### 2.3 index.html

```html
...
  <body>
    <div id="name-of-micro-app" data-attribute-1="name-of-micro-app666"></div>
    <script type="module" src="/src/main.tsx"></script>
  </body>
...
```
_For local testing and development purposes._


## 3. Build
Specify output files name: 
```js
// vite.config.ts

    rollupOptions: {
…
+     output: {
+       assetFileNames: 'assets/[name][extname]',
+       entryFileNames: '[name].js',
+     }
    }
…
```


## 4. Deploy
Can be deployed on Vercel via GitHub.

## 5. Injection of App
You will need add to the Host App:
### 5.1 HTML div which will accommodate your app and its script:
```html
<!-- index.html -->

...
<div id="name-of-micro-app" data-attribute-1="name-of-micro-app666"></div>
<script src="widget/index.js"></script>
...
```
### 5.2 Styles
1. Option - link
    ```html
    <!-- index.html // Header -->
    
    <link rel="stylesheet" href="https://calendar-mu-nine.vercel.app/assets/index.css">
    ```
2. Option - inline
    ```html
    <!-- index.html // Header -->
    
    <style>
    ...
    </style>
    ```

## 6. Examples of implementation in Host App
### 6.1 Rendering in Framer / React
```js
// MyComponent.tsx

import * as React from "react"

/**
 * These annotations control how your component sizes
 * Learn more: https://www.framer.com/developers/#code-components-auto-sizing
 *
 * @framerSupportedLayoutWidth any
 * @framerSupportedLayoutHeight any
 */

export default function TestCalendarWidget(props) {
    const jsSrc = `https://calendar-mu-nine.vercel.app/assets/index.js`
    const attr = `name-of-your-app`
    const widget = `<div id="name-of-micro-app" data-attribute-1=${attr}></div>`

    React.useEffect(() => {
        const wrap = document.createElement("div")
        const scr = document.createElement("script")
        scr.src = jsSrc
        scr.type = "text/javascript"
        wrap.appendChild(scr)
        document.getElementById("name-of-micro-app")?.appendChild(wrap)
    }, [])

    return (
        <div>
            <div dangerouslySetInnerHTML={{ __html: widget }} />
        </div>
    )
}
```


### 6.2 Rendering in HTML
```html
<!-- index.html -->

<html lang="en">
  <head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
        <link rel="stylesheet" href="https://calendar.vercel.app/assets/index.css">
  </head>
  <body>
    
    <div id="name-of-micro-app" data-attribute-1="name-of-micro-app79"></div>
    <script type="module" src="https://calendar.vercel.app/assets/index.js"></script>
  </body>
</html>
```








