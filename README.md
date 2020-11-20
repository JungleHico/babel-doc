# Babel

Babel 是一个 JavaScript 的编译器，可以将ES6的代码转化为浏览器支持的 JavaScript 代码（通常是 ES5），从而在现有环境中运行。这就意味着，我们可以使用 ES6 编写程序，而不依赖于浏览器是否支持。

Babel 官方文档地址：[https://www.babeljs.cn/](https://www.babeljs.cn/)

## 安装

1.新建项目，并在目录下初始化 npm（如果没有）：
```bash
npm init -y
```

2.安装 Babel（7+）：
```bash
npm install --save-dev @babel/core @babel/cli @babel/preset-env
```

* @babel/core，Babel 编译的核心库
* @babel/cli，Babel 的命令行工具，支持使用命令行进行编译
* @babel/preset-env，Babel 预置插件集，避免一个个插件进行配置

> 注意：Babel7 以后的包名以 `@babel/*` 的开头

## 配置

创建配置文件 `babel.config.json`：
```json
{
    "presets": [
        [
            "@babel/env",
            {
                "debug": true
            }
        ]
    ]
}
```

上面的配置使用 `@babel/presets-env` 作为预设的插件集，这个插件集包含了 ES6 的常用语法。为了方便调试，我们启用 `debug`，编译的时候就能打印出使用的插件。

## 运行

首先创建 `src` 目录，并创建 `src/index.js`，使用ES6的箭头函数语法：
```js
let f = (x) => x + 1;
```

在 `package.json` 的 `scripts` 中添加命令：

```json
{
    "scripts": {
        "babel": "babel src --out-dir dist"
    }
}
```

安装了 `@babel/cli`，我们可以通过命令行运行Babel，这里将 `src` 目录下的 JS 文件编译后，输出到 `dist` 目录，运行命令：

```bash
npm run babel
```

可以看到命令行打印了很多信息，并且生成了编译后的文件 `dist/index.js`：
```js
"use strict";

var f = function f(x) {
  return x + 1;
};
```

## @babel/preset-env 常用配置

参考官方文档：[https://www.babeljs.cn/docs/babel-preset-env](https://www.babeljs.cn/docs/babel-preset-env)

### targets
`string | Array<string> | { [string]: string }`，默认为`{}`

指定目标环境，`@babel/preset-env` 只转译目标环境不支持的语法，默认为空，即不考虑目标环境，这时 `@babel/preset-env` 就会转译所有的ES6语法。这个属性常用的值：
```json
{
    "presets": [
        [
            "@babel/env",
            {
                "debug": true,
                "targets": "> 1%, last 2 versions, not ie <= 8"
            }
        ]
    ]
}
```

该值的含义以及更多浏览器配置参考：[https://github.com/browserslist/browserslist](https://github.com/browserslist/browserslist)

### modules

### debug 

`Boolean`，默认为 `false`

是否打印使用的插件，之前的例子已经使用过。

### useBuiltIns

`"entry" | "usage" | false`，默认为 `false`

该参数控制 `polyfill` 的注入。`polyfill` 是一段代码，用来为旧浏览器提供它没有原生支持的较新的功能，可以称之为“垫片”或者“补丁”。

ES6可以分为新语法和新的API，`let`、`const`、箭头函数，解构赋值等都属于新语法，新增的对象或者新增的方法属于新的API，例如 `Promise` 对象，`Array.includes()` 方法。Babel默认只会转换新语法，不会转换新的API，新的API一般是通过引入 `polyfill` 来兼容环境。

旧版本的Babel使用 `@babel/polyfill` 来转译新的API，Babel7.4以后使用 `core-js`。安装 `core-js`（注意使用`--save`）：

```bash
npm install core-js --save
```

`useBuiltIns` 的可选值包括：`"entry"`、`"usage"` 和 `false`。

#### `useBuiltIns: "entry"`

指定为 `"entry"`，编译时，Babel会把目标环境不支持的ES6新的API全部注入，同时也需要在入口文件导入 `core-js`，修改之前的 `src/index.js`：

```js
import 'core-js';

import 'core-js';

let f = (x) => x + 1;

let p = new Promise();

const str = 'Hello Babel';
str.includes('Babel');
```

修改配置文件 `babel.config.json`：
```js
{
    "presets": [
        [
            "@babel/env",
            {
                "debug": true,
                "targets": "> 1%, last 2 versions, not ie <= 8",
                "useBuiltIns": "entry"
            }
        ]
    ]
}
```

运行 `npm run babel`，查看编译后的 `dist/index.js`：

```js
"use strict";

require("core-js/modules/es6.array.copy-within");

require("core-js/modules/es6.array.fill");

require("core-js/modules/es6.array.find");

// ...

require("core-js/modules/es6.promise");

// ...

require("core-js/modules/es6.string.includes");

// ...

var f = function f(x) {
  return x + 1;
};

var p = new Promise();
var str = 'Hello Babel';
str.includes('Babel');
```

编译后导入非常多的包，其中包括 `Promise` 和 `String.Prototype.includes()`，但是像上面的例子，导入许多不需要用的包，是不必要的，我们希望按需导入，这就可以通过设置 `useBuiltIns`的值为 `"usage"`。

#### `useBuiltIns: "usage"`

设置为 `usage`，不需要手动导入 `core-js`，编译时会按需注入。

编译后，只导入了相关的包：

```js
"use strict";

require("core-js/modules/es6.string.includes");

require("core-js/modules/es6.promise");

require("core-js/modules/es6.object.to-string");

var f = function f(x) {
  return x + 1;
};

var p = new Promise();
var str = 'Hello Babel';
str.includes('Babel');
```

### corejs

`2 | 3 | { version: 2 | 3, proposals: Boolean }`， 默认为 `2`

`core-js` 的版本，当 `useBuiltIns` 指定为 `"entry"` 或者 `"usage"`时，需要安装`core-js` 并且设置 `core-js` 的版本，否则会出现警告。修改 `babel.config.json`：

```json
{
    "presets": [
        [
            "@babel/env",
            {
                "debug": true,
                "targets": "> 1%, last 2 versions, not ie <= 8",
                "useBuiltIns": "usage",
                "corejs": 3
            }
        ]
    ]
}
```

## @babel/plugin-transform-runtime

开发第三方类库时用到，避免全局污染，参考官方文档：[https://www.babeljs.cn/docs/babel-plugin-transform-runtime](https://www.babeljs.cn/docs/babel-plugin-transform-runtime)

## 在Webpack中使用Babel

在实际项目开发中，我们很少直接使用 Babel，往往是结合 Webpack 等构建工具一起使用。事实上，如果项目中用到 ES6 新的 API，Babel 转义后导入的包来自 Node，无法在浏览器直接使用。

### 1. 初始化Webpack

Webpack 的配置参考官方文档：[https://www.webpackjs.com/concepts/](https://www.webpackjs.com/concepts/)

### 2.安装Babel

```
npm install --save-dev babel-loader @babel/core @babel/preset-env
```

相比之前的配置，多安装了 `babel-loader`，去除了 `@babel/cli`，我们不再通过 Babel 的命令行编译，而是在 Webpack 打包时，识别JS文件，通过 `babel-loader`，将其中的代码转译成浏览器支持的代码。

修改 Webpack 的配置文件：
```js
const path = require('path');
const HtmlWebpackPlugin = require('html-webpack-plugin');

module.exports = {
    entry: './src/index.js',
    output: {
        filename: 'bundle.js',
        path: path.resolve(__dirname, 'dist')
    },
    module: {
        rules: [
            // ...
            {
                test: /\.js$/,
                exclude: /(node_modules|bower_components)/,    // 排除 node_modules 目录
                use: {
                    loader: 'babel-loader'
                }
            }
        ]
    },
    plugins: [
        new HtmlWebpackPlugin({
            template: 'index.html'
        })
    ]
};
```

### 3.配置Babel

创建 `.babelrc` 或者 `.bable.config.json`：

```json
{
    "presets": [
        [
            "@babel/env",
            {
                "debug": true,
                "targets": "> 1%, last 2 versions, not ie <= 8",
                "useBuiltIns": "usage",
                "corejs": 3
            }
        ]
    ]
}
```

可以不指定 `@babel-preset-env` 的 `targets` 值，而是在 `package.json` 中指定 `browserslist`。

除了添加 Babel 的配置文件，也可以在 Webpack 的配置文件中进行配置（不推荐），参考：[https://webpack.js.org/loaders/babel-loader/](https://webpack.js.org/loaders/babel-loader/)

### 4.运行Webpack命令
