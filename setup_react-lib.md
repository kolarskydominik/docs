# Create and Publish a Component Library (NPM)
sources:
- https://www.pluralsight.com/guides/react-typescript-module-create
- https://www.freecodecamp.org/news/how-to-create-and-publish-your-first-npm-package/
- https://dev.to/receter/how-to-create-a-react-component-library-using-vites-library-mode-4lma
- https://dev.to/receter/the-minimal-setup-to-package-and-reuse-your-react-components-1063#publishinstall-your-package

## React + Vite + TypeScript + CSS.Modules

### 1. Setup a new Vite project

```shell
npm create vite@latest
? Project name: › my-component-library
? Select a framework: › React
? Select a variant: › TypeScript
cd my-component-library
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

### 2. Basic build setup
```
 📂my-component-library
 +┣ 📂lib
 +┃ ┗ 📜main.ts
  ┣ 📂public
  ┣ 📂src
  …
```
_Think of all code inside the src folder as your demo page._

The main entry point of your library will be a file named **main.ts** inside of **lib**.

#### Vite Library Mode
To activate this mode, simply specify your library entry point in vite.config.ts.

```js
//vite.config.ts

import { defineConfig } from 'vite'
+ import { resolve } from 'path'
import react from '@vitejs/plugin-react'

export default defineConfig({
  plugins: [react()],
+  build: {
+    lib: {
+      entry: resolve(__dirname, 'lib/main.ts'),
+      formats: ['es']
+    }
  }
})
```
📘 Library mode docs - https://vitejs.dev/guide/build.html#library-mode

📘 lib mode docs - https://vitejs.dev/config/build-options.html#build-lib

#### TypeScript and library mode

```js
//tsconfig.json

-   "include": ["src"],
+   "include": ["src", "lib"],

```
_To enable Typescript for your newly created lib folder._


```
 📂my-component-library
  ┣ …
  ┣ 📜tsconfig.json
 +┣ 📜tsconfig-build.json
  …
```

```js
//tsconfig-build.json

{
  "extends": "./tsconfig.json",
  "include": ["lib"]
}
```

```js
//package.json

 "scripts": {
    …
-   "build": "tsc && vite build",
+   "build": "tsc --p ./tsconfig-build.json && vite build",
```
_The only difference between the default configuration and the build configuration is that for building you only have lib included instead of lib and src._

Copy the file **vite-env.d.ts** from **src** to **lib**. Without this file Typescript will miss some types definitions provided by Vite when building (because we don't include src anymore).

```js
//vite.config.ts

build: {
+  copyPublicDir: false,
…
}
```
_Disable Vite default of copying stuff from public folder to dist._

#### Building the types
```shell
npm i vite-plugin-dts -D
```

```js
// vite.config.ts

+import dts from 'vite-plugin-dts'
…
  plugins: [
    react(),
+   dts({ include: ['lib'] })
  ],
…
```

### 3. Exclude React from build
You can try:
```
 📂my-component-library
  ┣ 📂lib
 +┃ ┣ 📂components
 +┃ ┃ ┣ 📂Button
 +┃ ┃ ┃ ┗ 📜index.tsx
  ┃ ┗ 📜main.ts
  …
```

```js
// lib/components/Button/index.tsx

export function Button(props: React.ButtonHTMLAttributes<HTMLButtonElement>) {
  return <button {...props} />
}
```

```js
// lib/main.ts

export { Button } from './components/Button'
```
Build: ~77kb

Remove React from buil:
```js
//vite.config.ts

  build: {
    …
+   rollupOptions: {
+     external: ['react', 'react/jsx-runtime'],
+   }
  }
```
Build: 0.14kb

### 4. Add styles
```
 📂my-component-library
  ┣ 📂lib
  ┃ ┣ 📂components
  ┃ ┃ ┣ 📂Button
  ┃ ┃ ┃ ┣ 📜index.tsx
+ ┃ ┃ ┃ ┗ 📜styles.module.css
  ┃ ┗ 📜main.ts
  …
```

```css
/* lib/components/Button/styles.module.css */
.button {
    padding: 1rem;
}
```

```js
// lib/components/Button/index.tsx

import styles from './styles.module.css'

export function Button(props: React.ButtonHTMLAttributes<HTMLButtonElement>) {
  const { className, ...restProps } = props
  return <button className={`${className} ${styles.button}`} {...restProps} />
}
```
#### Keep import of the CSS
```shell
npm i vite-plugin-lib-inject-css -D
```

```js
// vite.config.ts

+import { libInjectCss } from 'vite-plugin-lib-inject-css'
…
  plugins: [
    react(),
+   libInjectCss(),
    dts({ include: ['lib'] })
  ],
…
```
try npm run build

#### Split up the CSS to the chunks
_One way of doing this would be to turn every file into an Rollup entry point. And, it couldn't be better, there is a recommended way of doing this right in the Rollup documentation: https://rollupjs.org/configuration-options/#input_

```shell
npm i glob -D
```
Specify input files name:
```js
// vite.config.ts

-import { resolve } from 'path'
+import { extname, relative, resolve } from 'path'
+import { fileURLToPath } from 'node:url'
+import { glob } from 'glob'
…
    rollupOptions: {
      external: ['react', 'react/jsx-runtime'],
+     input: Object.fromEntries(
+       glob.sync('lib/**/*.{ts,tsx}').map(file => [
+         // The name of the entry point
+         // lib/nested/foo.ts becomes nested/foo
+         relative(
+           'lib',
+           file.slice(0, file.length - extname(file).length)
+         ),
+         // The absolute path to the entry file
+         // lib/nested/foo.ts becomes /project/lib/nested/foo.ts
+         fileURLToPath(new URL(file, import.meta.url))
+       ])
+     )
    }
…
```
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

### 5. Last steps before publish
#### Main file
Your library's primary entry point is now located at **dist/main.js**, so this needs to be set in your **package.json**. The same applies to the type's entry point: **dist/main.d.ts**

```js
// package.json

{
  "name": "my-component-library",
  "private": true,
  "version": "0.0.0",
  "type": "module",
+ "main": "dist/main.js",
+ "types": "dist/main.d.ts",
  …
```

#### Define the files to publish
```js
// package.json

  …
  "main": "dist/main.js",
  "types": "dist/main.d.ts",
+ "files": [
+   "dist"
+ ],
  …
```

#### Dependencies
```js
// package.json

- "dependencies": {
+ "peerDependencies": {
    "react": "^18.2.0",
    "react-dom": "^18.2.0"
  },
  "devDependencies": {
+   "react": "^18.2.0",
+   "react-dom": "^18.2.0",
    …
  }
```

#### Side effects
To prevent the CSS files from being accidentally removed by the consumer's tree-shaking efforts, you should also specify the generated CSS as side effects:

```js
// package.json

+ "sideEffects": [
+   "**/*.css"
+ ],
```

#### Ensure that the package is built
```js
// package.json

  "scripts": {
    "dev": "vite",
    "build": "tsc && vite build",
    …
+   "prepublishOnly": "npm run build"
  },
```

### 6. Publish
To publish to my user scope on npm
```js
-  "name": "my-components",
+  "name": "@my-name/my-components",
+  "private": false,
```


First publish
```shell
npm publish --access public
```

Before any other publish, you need update version
```shell
npm version patch
npm publish
```













