## 项目构建
- 新建项目文件夹
- npm init
- 设置项目目录结构，结构如下：
![image.png](http://upload-images.jianshu.io/upload_images/659084-bd9163add73e9ea0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- **build** 目录是 webpack 打包后的生成目录，**index.html** 是打包后的 html 文件，bundle.hash.js 是打包后的 js 文件。
- **src** 目录是项目源文件，**template.html** 是 html 文件模板。
- **webpack.config.js** 是配置 **webpack** 的总文件。

```
module.exports = { ... }
```

## 模式 mode

```
mode: "development", // development or production
```

## 入口 entry
设置打包的入口文件, entry: filepath

```
entry: "./src/index.js", // 入口
```

## 输出 output
设置打包后的输出文件，output: { ... }

```
output: { // 输出
  filename: "bundle.[hash:8].js",
  path: path.resolve(__dirname + '/build'),
  // publicPath: "http://", // 公共路径前缀
},
```
注：
- **path** 后面应是绝对路径。
- **bundle.[hash:8].js** 中的 **[hash:8]** 可以在每次打包后都在文件后面追加 hash 值。

通过上面三项，可以实现一个简单的 **webpack** 打包配置。在 **package.json** 添加脚本或直接执行 **./node_modules/.bin/webpack** 即可实现打包：

```
"build": "./node_modules/.bin/webpack",
```

## webpack-dev-server
webpack 开发服务器，用于开发时候的配置：
- yarn add taco --dev webpack-dev-server / npm install --save-dev webpack-dev-server
- 基本配置如下：

```
devServer: { // 开发服务器
  port: 3001,
  contentBase: path.resolve(__dirname + '/build'), // 本地服务器目录
  progress: true, // 进度条
  open: true, // 自动打开浏览器
  compress: true, // 压缩
  // 1. 代理
  proxy: {
    "/api": {
      target: "",
      pathRewrite: {
        "/api": ""
      }
    }
  },
  // 2. 用 express 内置钩子模拟数据
  before(app) {
    app.get("/api", (req, res) => { ... })
  },
  // 3. 在服务端中启动 webpack 服务，使用 webpack-dev-middleware 中间件
},
},
```

- 在 **package.json** 添加脚本即可：

```
"dev": "webpack-dev-server"
```

## source-map && eval-source-map && cheap-module-source-map && cheap-module-eval-source-map

作用：源码映射，报错时显示出错的位置。

- source-map：会单独生成一个 **sourcemap** 文件；会显示行列。
- eval-source-map：不会单独生成一个 **sourcemap** 文件，会将生成的 **sourcemap** 放到打包后的 **html** 文件；会显示行和列。
- cheap-module-source-map：会生成 **sourcemap** 文件，不会显示列。
- cheap-module-eval-source-map：不会生成 **sourcemap** 文件，集成在 **html** 文件中，不会显示列。

```
devtool: "source-map", // 源码映射
```

## watch
作用：监控代码实时变化，进行编译打包。

```
watch: true, // 监控代码实时变化，进行编译打包
watchOptions: {
  poll: 1000, // 多毫秒监控一次
  aggreatement: 500, // 防抖
  ignored: "node_modules" // 忽略文件夹
},
```

## resolve
作用：解析第三方包

```
resolve: { // 解析 第三方包
  modules: [
    path.resolve("node_modules"), // 在当前目录下查找模块 
    path.resolve("other_modules"), // 在其他目录下查找模块 
  ],
  // 自动添加扩展名，主要是 import 时候使用, 依次解析
  extensions: [".js", ".css", ".json", ".vue"],
  // 比如引用 bootstrap 的 css 样式
  // 方式一： bootstrap 的主入口
  mainFields: ["style", "main"],
  // 方式二：别名  
  // alias: {
  //   bootstrap: "bootstrap/dist/css/bootstrap.css"
  // }
},
```

## 插件
webpack 的插件配置是一个数组，里面存放着各种各样的插件。

```
plugins: [
 new pluginName(...)
]
```

#### html-webpack-plugin
插件作用：打包时候自动根据 **html** 模板生成目标 **html** 文件，自动生成目标打包目录；另外，可配置 **html** 的相关打包配置，如：压缩，去双引号等。

```
new HtmlWebpackPlugin({
  filename: "index.html", // 目标文件名称
  template: path.resolve(__dirname + '/src/template.html'), // 模板文件
  minify: { // 压缩配置
    removeAttributeQuotes: true, // 去双引号
    collapseWhitespace: true, // 不换行
  },
  hash: true // 生成 hash 戳
})
```

#### mini-css-extract-plugin
作用：将生成的 **css样式** 抽离成一个 **css文件**，并将该样式文件引进目标 **html** 文件中。

```
plugins: [ // 插件
  new MiniCssExtractPlugin({
    filename: "main.css",
  })
],
module: { // 模块
  rules: [{
    test: /\.css$/,
    use: [
      // {
      //   loader: "style-loader",
      //   options: {
      //     insertAt: "top" // 插到 head 标签的顶部，更改优先级
      //   }
      // },
      MiniCssExtractPlugin.loader, // 注意 loader 的使用顺序
      "css-loader"
    ]
  }]
}
```

注：这里抽离出来的 **main.css** 文件并没有压缩，要通过手动添加以下两个插件到优化项：
- optimize-css-assets-webpack-plugin
- uglifyjs-webpack-plugin

```
optimization: { // 优化项
  minimizer: [
    new OptimizeCssAssetsWebpackPlugin({}), // 压缩抽离的 css 文件
    new UglifyjsWebpackPlugin({
      cache: true,
      parallel: true,
    }) // 使用了 optimize-css-assets-webpack-plugin 插件后，原先的 js 压缩不起作用，要手动重新配置
  ]
},
```

#### optimize-css-assets-webpack-plugin
插件作用：压缩抽离出来的 **main.css** 文件。但是，使用了 **optimize-css-assets-webpack-plugin** 插件后，原先的 js 压缩不起作用，要手动重新配置优化项 **uglifyjs-webpack-plugin** 。

#### uglifyjs-webpack-plugin
插件作用：压缩打包后的 **js** 文件。

#### clean-webpack-plugin
作用：清理文件

```
plugins: [
 new CleanWebpackPlugin("./build")
]
```

#### copy-webpack-plugin
作用：将某些文件拷贝到打包后的文件夹。

```
new CopyWebpackPlugin([
  {from: "./copy", to: "./"}
])
```

#### banner-plugin
作用：版权声明，是 **webpack** 的内置模块。

```
new webpack.BannerPlugin("@copyright by Ertsul")
```

#### webpack.DefinePlugin({ ... })
作用：定义环境变量。

```
plugins: [
  new webpack.DefinePlugin({
    DEV: JSON.stringify("development"),
    PRO: JSON.stringify("production")
  })
]
```

#### webpacfk.IgnorePlugin()
作用：忽略模块的引进

```
plugins: [
  new webpack.IgnorePlugin(/\./locale/, /moment/)
]
```

注：如果直接使用 **DEV: 'development'** 的话，会把 **DEV** 直接替换为 **引号内部的内容**。如：**console.log(DEV)**，会变成 **console.log(dev)**，最后的结果是 **undefined**。所以，需要通过 **JSON.stringfy()** 进行转化。

## 模块 module
模块主要是各种 **loader**，作用：解析各种类型的文件。

```
module: { 
 rules: [
   {
     test: regx,
     use: [loadName]
   }
 ]
}
```

#### css-loader / style-loader
- css-loader：主要是用于解析在 **css** 文件中通过 **@import** 方式引进其他的 **css** 文件。
- style-loader: 主要是用于将 **js** 文件中通过 **require** 方式引进的 **css** 文件插入到目标 **html** 文件的 **head** 中（插到最后，层级最高）。

```
// 数组方式
module: { // 模块
  rules: [{
    test: /\.css$/, // 正则
    use: ["style-loader", "css-loader"] // loader 数组
  }]
}
// 对象方式
module: { // 模块
  rules: [{
    test: /\.css$/, // 正则
    use: [
      {
        loader: "style-loader",
        options: {
          insertAt: "top" // 插到 head 标签的顶部，更改优先级
        }
      }, 
      "css-loader"
    ]
  }]
}
```
注：**loader** 是从右向左、从下到上的执行顺序，故：**use: ["style-loader", "css-loader"]**

#### postcss-loader + autoprefixer
这两个的配合使用可以自动添加 **css** 的浏览器前缀。
- autoprefixer：自动添加 **css** 的浏览器前缀。
- postcss-loader：loader 处理 autoprefixer。

```
// 使用 postcss-loader 
module: { // 模块
  rules: [{
    test: /\.css$/,
    use: [
      // {
      //   loader: "style-loader",
      //   options: {
      //     insertAt: "top" // 插到 head 标签的顶部，更改优先级
      //   }
      // },
      MiniCssExtractPlugin.loader,
      "css-loader",
      "postcss-loader" // 注意顺序
    ]
  }]
}
```

- 根目录下新建 postcss.config.js 文件

```
module.exports = {
  plugins: [
    require("autoprefixer")
  ]
}
```

#### babel-loader + @babel/core + @babel/preset-env
作用：转化 es6 语法
  
```
module: { // 模块
  rules: [{
    test: /\.js$/,
    use: {
      loader: "babel-loader",
      options: { // 用 babel-loader 需要把 es6 转化为 es5
        presets: [
          "@babel/preset-env", // 转化 js 语法，一个大的 babel 库，包含将 es6 转化为 es5 的模块
        ],
        // plugins: [...] // 其他的小插件
      }
    }
  }]
}
```

#### expose-loader
作用：暴露全局的 **loader**，暴露到 **window** 上。

```
// 一般情况
import $ from 'jquery';
console.log(window.$); // undefined
// 内联 loader 方式
import $ from 'expose-loader?$!jquery';
console.log(window.$);
// 其他方式：在每个模块中注入
new webpack.ProvidePlugin({
  $: "jquery"
})
console.log($);
console.log(window.$); // undefined
```

#### file-loader && url-loader && html-withimg-loader
在 **webpack** 项目中不能直接使用相对路径的方式调用图片，不然打包后的图片路径会报错。

- file-loader：**js** 文件中通过 **import** 方式引进图片。
- html-withimg-loader：**html** 文件中通过 **scr** 方式引进图片。
- 在 **css** 中使用图片，**style-loader** 已经做了处理。
- url-loader：将图片转化为 **base64**

```
module: {
  rules: [
    {
      test: /\.(png|jpg|gif)$/,
      use: {
        loader: "url-loader",
        options: {
          limit: 200 * 1024, // 图片小于 200k，变成 base64，否则用 file-loader
          // publicPath: "http://", // 公共路径前缀
        }
      }
    },
    {
      test: /\.html$/,
      use: ["html-withimg-loader"]
    }
  ]
}
```
`
## 模块 module -- noParse
作用：不解析指定包的依赖关系。

```
module: {
  noParse: /jquery/
}
```

## 模块 module -- exclude
作用：排除

```
module: {
  exclude: /node_module/
}
```

## 模块 module -- include
作用：包含

```
module: {
  include: /src/
}
```



## 区分不同环境，实现开发和生产配置的分离
将 **webpack** 的配置文件划分为以下文件：

- webpack.base.js：基本、共有的配置。
- webpack.dev.js：开发配置。
- webpack.prod.js：生产配置。

使用 **webpack-merger** 插件：
- 安装 **webpack-merge** 插件；
- 引进目标模块

```
import { smart } from "webpack-merge";
```

- 使用

```
const base = require("webpack.base.js);

module.exports = smart(base, {
  ...
})
```

## webpack 自带优化
- **import** 语法（生成环境），**tree-shaking** 自动删除没有用到的代码；
- **es6** 模块会把语法放到 **default** 上；
- 自动省略可以简化的代码；
- **scope hosting**，作用域提升。

## webpack 懒加载
- 通过 **import** 实现（该语法内部由 **jsonp** 实现），返回的是一个 **promise**。
- 要在 **babel-loader** 中添加 **@babel/plugin-syntax-dynamic-import** 插件。
- **vue** 和 **react** 的l懒加载都是这样实现的。

```
/*
* 模拟点击加载 source.js 的内容
*/
// 1
{
  loader: 'babel-loader',
  options: {
    presets: [
      '@babel/preset-env'
    ],
    plugins: [
      '@babel/plugin-syntax-dynamic-import'
    ]
  },
}
// 2
let btn = document.createElement('button');
btn.innerHTML = 'btn';
btn.addEventListener('click', function () {
  // 内部由 jsonp 实现动态加载文件
  import('./source.js')
    .then(data => {
      console.log(data);
    })
})
document.body.appendChild(btn);

```

点击按钮后结果如下：
![image.png](http://upload-images.jianshu.io/upload_images/659084-fa79ce43578285a0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 热更新 / 热替换 / Hot Module Replacement / HMR
热更新：页面只更新改动的模块。

- **devServer** 开始 **hot** 热更新模块。
- 使用 **webpack** 内置热更新插件。
  - NamedModulesPlugin
  - HotModuleReplacementPlugin

```
// devServer 配置
devServer: {
  port: 8089,
  contentBase: path.resolve(__dirname + '/dist/'),
  compress: true,
  progress: true,
  // open: true
  hot: true, // 开启热更新，只更新更改的模块
},
// 添加插件
plugins: [
  new webpack.NamedModulesPlugin(), // 热更新打印更新的模块路径
  new webpack.HotModuleReplacementPlugin(), // 热更新
]
```

- 使用热更新

```
import source from './source.js';

console.log(source);

// 添加热更新操作，不然不会实现热更新
if (module.hot) {
module.hot.accept('./source.js', () => {
  require('./source.js');
})
```

## 多线程打包
使用 **happypack** 可以实现多线程打包。

- 安装并引进 **happypack**;
- 改写 **module**;
- **plugin** 配置。

```
// 改写 module
module: {
  rules: [
    {
      test: /\.js$/,
      // use: {
      //   loader: 'babel-loader',
      //   options: {
      //     presets: [
      //       '@babel/preset-env'
      //     ]
      //   }
      // }
      use: {
        loader: 'Happypack/loader?id=js',
      }
    },
  ]
}
// plugin 配置
plugins: [
  new Happypack({
    id: 'js',
    use: [
      {
        loader: 'babel-loader',
        options: {
          presets: [
            '@babel/preset-env'
          ]
        }
      }
    ]
  }),
]
```

## 多页面打包

```
const path = require("path");
const HtmlWebpackPlugin = require("html-webpack-plugin");

module.exports = {
  mode: "production",
  entry: {
    home: "./src/home.js",
    about: "./src/about.js",
    other: "./src/other.js"
  },
  output: {
    path: path.resolve(__dirname + "/build/"),
    filename: "[name].js",
  },
  plugins: [
    new HtmlWebpackPlugin({
      template: path.resolve(__dirname + "/src/template.html"),
      filename: "home.html",
      chunks: ["home"]
    }),
    new HtmlWebpackPlugin({
      template: path.resolve(__dirname + "/src/template.html"),
      filename: "other.html",
      chunks: ["other"]
    }),
    new HtmlWebpackPlugin({
      template: path.resolve(__dirname + "/src/template.html"),
      filename: "about.html",
      chunks: ["about"]
    }),
  ]
}
```

## 多页面打包抽离公共代码块
打包 **多页面** 需要将公共的代码块抽离出来进行优化：

```
optimation: {
  splitChunks: {  // 分割代码块
    cacheGroups: {  // 缓存组
      common: {  // 公共的模块
        chunks: 'initial',  // 从哪里开始，initial 即：入口
        miniSize: 0,  // 大小大于多少字节的时候抽离
        miniChunks: 2  // 引用多少次的时候抽离
      },
      vender: {  // 第三方模块
        priority: 1,  // 设置权重，没有设置权重的话，代码默认从上到下执行，上面的公共代码会先将第三方模块抽离出来，达不到抽离第三方模块的作用
        test: /node_modules/,  // 抽离出来
        chunks: 'initial',  // 从哪里开始，initial 即：入口
        miniSize: 0,  // 大小大于多少字节的时候抽离
        miniChunks: 2  // 引用多少次的时候抽离
      }
    }
  }
}
```

## Tapable
**webpack** 本质是事件流的机制，工作流程就是将各个插件串联起来，实现核心就是 **Tapable**，通过 **Tapable** 实现各种钩子（如同步钩子异步钩子）。而 **Tapable** 核心是依赖于 **发布订阅者模式**。

- 同步钩子基本思路（发布订阅者模式）：将所有注册的函数存放到一个任务数组，发布的时候所有订阅者都会执行。使用 **tap** 注册。

使用：
```
const { SyncHook } = require('tapable');

class Hook {
  constructor() {
    this.hooks = {
      arch: new SyncHook(['name'])
    }
  }
  tap() { // 注册监听事件
    this.hooks.arch.tap('vue', (name) => {
      console.log('vue', name);
    })
    this.hooks.arch.tap('react', (name) => {
      console.log('react', name);
    })
  }
  start() { // 开始
    this.hooks.arch.call('Ertsul')
  }
}
let h = new Hook();
h.tap();
h.start();
```

- 异步钩子基本思路（发布订阅者模式）：将所有注册的函数存放到一个任务数组，发布的时候所有订阅者都会执行。使用 **tapAsync** 注册。

setTimeout()方式使用：

```
const { AsyncParallelHook } = require('tapable');

class AsyncHook {
  constructor() {
    this.hooks = {
      arch: new AsyncParallelHook(['name'])
    }
  }
  tap() {
    this.hooks.arch.tapAsync('vue', (name, cb) => {
      setTimeout(() => {
        console.log('vue', name);
        cb && cb();
      }, 1000)
    })
    this.hooks.arch.tapAsync('react', (name, cb) => {
      setTimeout(() => {
        console.log('react', name);
        cb && cb();
      }, 1000)
    })
  }
  start() {
    this.hooks.arch.callAsync('Ertsul', () => {
      console.log('All finished!');
    })
  }
}

const a = new AsyncHook();
a.tap();
a.start();
```

Promise()方式使用：
```
const { AsyncParallelHook } = require('tapable');

class AsyncHook {
  constructor() {
    this.hooks = {
      arch: new AsyncParallelHook(['name'])
    }
  }
  tap() {
    this.hooks.arch.tapPromise('vue', (name) => {
      return new Promise((resolve, reject) => {
        setTimeout(() => {
          console.log('vue', name);
          resolve();
        }, 1000)
      })
    })
    this.hooks.arch.tapPromise('react', (name) => {
      return new Promise((resolve, reject) => {
        setTimeout(() => {
          console.log('react', name);
          resolve();
        }, 1000)
      })
    })
  }
  start() {
    this.hooks.arch.promise('Ertsul').then(() => {
      console.log('hook all finished!');
    })
  }
}

const a = new AsyncHook();
a.tap();
a.start();
```

#### Synchook 同步钩子
实现：
```
class Synchook {
  constructor(args) { // args => ['Synchook']
    this.tasks = []; // 注册事件数组
  }
  tap(name, task) { //  注册事件
    this.tasks.push(task);
  }
  call(...args) { // 发布事件
    this.tasks.forEach(task => {
      task(...args);
    })
  }
}

let hook = new Synchook(['Synchook']);
hook.tap('vue', (name) => {
  console.log('vue', name);
})
hook.tap('react', (name) => {
  console.log('react', name);
})
hook.call('Ertsul');
```
![image.png](http://upload-images.jianshu.io/upload_images/659084-1d12026d2f938c58.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### SyncBailHook 同步熔断钩子
Synchook 同步钩子在执行每个函数的时候会有一个返回值，undefined：执行成功，继续执行下一个函数，!= undefined：熔断执行，不执行下一个函数。

实现：
```
// Syncbailhook 同步熔断执行，会判断返回是否等于 undefined 来执行下一个函数
class Syncbailhook {
  constructor(args) {
    this.tasks = [];
  }
  tap(name, task) {
    this.tasks.push(task);
  }
  call(...args) {
    let ret = ''; // 当前函数的返回值
    let index = 0; // 当前任务数组的索引
    do {
      ret = this.tasks[index++](...args);
    } while (ret === undefined && index < this.tasks.length)
  }
}

let hook = new Syncbailhook(['Syncbailhook']);
hook.tap('vue', (name) => {
  console.log('vue', name);
})
hook.tap('react', (name) => {
  console.log('react', name);
})
hook.call('Ertsul');
```

![image.png](http://upload-images.jianshu.io/upload_images/659084-3a46a477c8b08d40.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### SyncWaterfallHook 同步瀑布钩子
Syncwaterfallhook 同步瀑布执行，各个函数之间存在关系，上一个函数的执行返回结果在当前函数的输入。

实现：
```
/**
 * Syncwaterfallhook 同步瀑布执行，各个函数之间存在关系
 * 上一个函数的执行返回结果在当前函数的输入
 */
class Syncwaterfallhook {
  constructor(args) {
    this.tasks = [];
  }
  tap(name, task) {
    this.tasks.push(task);
  }
  call(...args) {
    let [first, ...others] = this.tasks;
    let ret = first(...args);
    others.reduce((prev, current) => {
      return current(prev);
    }, ret)
  }
}

let hook = new Syncwaterfallhook(['Syncwaterfallhook']);
hook.tap('vue', (name) => {
  console.log('vue', name);
  return 'vue learnt';
})
hook.tap('react', (data) => {
  console.log('react', data);
})
hook.call('Ertsul');
```

![image.png](http://upload-images.jianshu.io/upload_images/659084-5ccbc856cac523cf.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### SyncLoopHook 同步循环钩子
同步循环钩子回让某个函数执行一定的次数。

实现：
```
// Tapable Syncloophook 同步循环钩子
class Syncloophook {
  constructor(args) { // args => ['Synchook']
    this.tasks = []; // 注册事件数组
  }
  tap(name, task) { //  注册事件
    this.tasks.push(task);
  }
  call(...args) { // 发布事件
    this.tasks.forEach(task => {
      let ret = '';
      do {
        ret = task(...args);
      } while (ret != undefined);
    })
  }
}

let hook = new Syncloophook(['Syncloophook']);
const total = 3;
let index = 0;
hook.tap('vue', (name) => {
  console.log('vue', name);
  return ++index === total ? undefined : 'continue';
})
hook.tap('react', (name) => {
  console.log('react', name);
})
hook.call('Ertsul');
```

![image.png](http://upload-images.jianshu.io/upload_images/659084-fd5d933715541d45.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### AsyncParallelHook 异步并发钩子 回调函数方式
实现：
```
// Tapable Asynchook 异步钩子
class asyncparallelhook {
  constructor(args) { // args => ['Synchook']
    this.tasks = []; // 注册事件数组
  }
  tapAsync(name, task) { //  注册事件
    this.tasks.push(task);
  }
  callAsync(...args) { // 发布事件
    let finalCallback = args.pop();
    let index = 0;
    const done = () => {
      index++;
      if (index === this.tasks.length) {
        finalCallback();
      }
    }
    this.tasks.forEach(task => {
      task(...args, done);
    })
  }
}

let hook = new asyncparallelhook(['asyncparallelhook']);
hook.tapAsync('vue', (name, cb) => {
  setTimeout(() => {
    console.log('vue', name);
    cb && cb();
  }, 1000)
})
hook.tapAsync('react', (name, cb) => {
  setTimeout(() => {
    console.log('react', name);
    cb && cb();
  }, 1000)
})
hook.callAsync('Ertsul', () => {
  console.log('All hook finished!');
});
```

#### AsyncParallelHook 异步并发钩子 Promise方式
实现：
```
// Tapable Asynchook 异步钩子
class asyncparallelhook {
  constructor(args) { // args => ['Synchook']
    this.tasks = []; // 注册事件数组
  }
  tapPromise(name, task) { //  注册事件
    this.tasks.push(task);
  }
  promise(...args) { // 发布事件
    let tasks = this.tasks.map(task => task(...args));
    return Promise.all(tasks);
  }
}

let hook = new asyncparallelhook(['asyncparallelhook']);
hook.tapPromise('vue', (name) => {
  return new Promise((resolve, reject) => {
    setTimeout(() => {
      console.log('vue', name);
      resolve();
    }, 1000)
  })
})
hook.tapPromise('react', (name) => {
  return new Promise((resolve, reject) => {
    setTimeout(() => {
      console.log('react', name);
      resolve();
    }, 1000)
  })
})
hook.promise('Ertsul').then(() => {
  console.log('All hooks finished!');
})
```

#### AsyncSeriesHook 异步串行钩子
#### AsyncWaterfallHook 异步瀑布流钩子