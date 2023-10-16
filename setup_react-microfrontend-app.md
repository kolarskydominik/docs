# Create and Deploy Micro-Frontend App to use in Host App (React+TS+Vite)

sources:
- https://dev.to/giveitatry/microfrontends-in-reactjs-do5
  
## React + Vite + TypeScript + CSS.Modules

### 1. Setup a new Vite project

```shell
npm create vite@latest
? Project name: › my-micro-app
? Select a framework: › React
? Select a variant: › TypeScript
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

