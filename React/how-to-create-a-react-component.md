# 手摸手教你发布一个 React 组件

![](https://img.shields.io/badge/status-Archive-brightgreen)

使用 TypeScript 和 webpack 搭建一个 react 组件的开发脚手架模板。

## 调试

使用 `yarn link` 或 `npm link` 来进行组件的调试。

## 目录

### example

用于进行功能展示和本地调试。

### es

用于存放编译完的代码，存放的代码为 ES5 语法的版本。模块使用的是 esnext。

### lib

用于存放编译完的代码，存放的代码为 ES5 语法的版本。因为我们开发的是一个 react 组件。所以直接使用 CommonJS 的模块即可，不需要将代码编译成 UMD 的格式。因为浏览器并没有内置 react 库你无法直接运行你的组件在没有 react 依赖的情况下。

### src

用于存放我们的源码。

## 依赖

### main

当使用者在使用你的组件时，它调用的入口文件就是你在 `main` 中指定的文件。通常我们会将它指向 `lib/index.js`。

### files

用于指定你发布的 `npm` 包内所包含的源码文件。一般情况下只要将编译完的代码发布出去即可，而组件本身的源码不需要包含在 `npm` 包中。使用者可以通过 `github` 来查看或调试该组件。

### dependencies

对于一个 `react` 组件而言，`react` 库的依赖其实是取决于你的宿主项目的。而在你的源码中 `react` 仅仅充当了一个开发调试的作用，所以将它放在 `devDependencies` 中，**你也可以将 `react` 组件理解为 `react` 库的一个插件，使用组件的前提是核心依赖库已经被正确安装**。同时你可以设定 `peerDependencies` 来提示使用者如何安装 `react` 的版本。

```json
{
  "main": "lib/index.js",
  "files": ["lib"],
  "devDependencies": {
    "@types/react": "^16.8.8",
    "@types/react-dom": "^16.8.3",
    "react": "^16.8.5",
    "react-dom": "^16.8.5",
    "typescript": "^3.3.4000"
  },
  "peerDependencies": {
    "react": ">=16.0.0"
  }
}
```

## tsconfig.json

我们可以使用 `npx tsc --init` 来初始化一个完整的 `typescript` 配置文件

- module：使用 `commonjs` 可以同时支持客户端渲染以及服务端渲染
- moduleResolution：当 `module` 设置 为 `commonjs` 时 `moduleResolution` 默认为 node
- jsx：我们不需要将 `.tsx` 文件编译成 `.jsx` 再做二次开发，所以直接编译成浏览器能够解析的 `.js` 文件即可
- declaration：可以自动生成声明文件，我们不用显示的去编写我们的声明文件

```json
{
  "compilerOptions": {
    "module": "commonjs",
    "moduleResolution": "node",
    "jsx": "react",
    "declaration": true,
    "declarationMap": true,
    "sourceMap": true,
    "outDir": "lib"
  }
}
```

## 参考

- [Create a React Component Library using TypeScript](https://www.youtube.com/watch?v=wcThWngBv7U)
- [浅谈 npm 的依赖与版本](https://github.com/SamHwang1990/blog/issues/7)
- [package.json 的 files 字段](https://docs.npmjs.com/files/package.json#files)
