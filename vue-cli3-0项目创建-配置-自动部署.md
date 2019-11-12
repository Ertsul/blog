## 创建项目
- 安装 **vue3.0** 脚手架
```
npm install -g @vue/cli
```

- 创建项目
```
vue create my_project
```
一系列选择之后，可以保存你当前的选择配置，之后创建项目的时候就可以直接用了。

- 安装依赖
```
cd my_project
```

最后的项目目录结构如下：

![image.png](http://upload-images.jianshu.io/upload_images/659084-5267ed577dcb2fc6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 运行项目
**vue3.0** 运行项目主要有两种方式：
- 命令行方式：
```
npm run serve
```

- UI视图方式，这也是3.0版本的一大亮点。通过UI视图方式，可以帮助我们很多事，如：运行项目、安装依赖、安装插件等。
```
vue ui
```

## webpack 配置
vue 已经帮我们配置了很多，额外自己又添加了一些配置；在根目录添加 **vue.config.js** 配置文件：

```
const path = require('path');
const px2rem = require('postcss-px2rem');
const VconsoleWebpackPlugin = require('vconsole-webpack-plugin');
const TerserWebpackPlugin = require('terser-webpack-plugin');
// 判断是否为生产环境
const isProduction = process.env.NNODE_ENV === 'production';
// 路径处理函数
const resolve = name => path.resolve(__dirname, name);

module.exports = {
  publicPath: isProduction ? '/production' : '/', // 部署应用包时的基本 URL
  productionSourceMap: !isProduction, // 生产环境不开启 source-map
  devServer: { // webpack-dev-server 的配置
    port: 8089, // 本地端口
    open: true, // 自动打开浏览器
    // proxy: { // 代理
    //   '/api': {
    //     target: '<url>',
    //     ws: true,
    //     changeOrigin: true
    //   },
    // }
  },
  css: {
    extract: isProduction, // 生产环境将 css 单独抽离成一个文件
    sourceMap: !isProduction, // 生成环境不开启 source-map
    loaderOptions: {
      postcss: {
        // 移动端使用rem
        plugins: [
          px2rem({
            remUnit: 75
          })
        ]
      },
      // 全局共享variables.scss
      sass: {
        // @/ 是 src/ 的别名
        // 所以这里假设你有 `src/variables.scss` 这个文件
        data: `@import "@/variables.scss";`
      }
    }
  },
  configureWebpack: config => {
    const devPlugins = [
      new VconsoleWebpackPlugin({ // 微信移动端调试控制台
        enable: !isProduction
      })
    ];
    const prodPlugins = [
      new TerserWebpackPlugin({
        terserOptions: { // 打包时候的配置
          warnings: false,
          compress: true,
          drop_console: true,
          drop_debugger: true,
          pure_funcs: ['console.log']
        }
      })
    ];
    if (isProduction) { // 生产环境
      config.optimization.minimizer = [
        ...config.optimization.minimizer,
        ...prodPlugins
      ]
    } else { // 开发环境
      config.plugins = [...config.plugins, ...devPlugins]
    }
  },
  chainWebpack: config => {
    // 自定义全局变量
    // config
    //   .plugin('define')
    //   .tap((args) => {
    //     args[0].PRODUCTION = isProduction ? JSON.stringify('') :
    //       JSON.stringify(
    //         'other/',
    //       );
    //     return args;
    //   });
    // 别名设置
    config.resolve.alias
      .set('js', resolve('src/assets/js'))
      .set('scss', resolve('src/assets/scss'))
      .set('images', resolve('src/assets/images'))
      .set('components', resolve('src/components'));
  }
}
```

## 路由配置
**vue3.0** 的路由配置已经很完善了，这里我仅仅小改动，封装了路由懒加载函数：

```
import Vue from 'vue'
import Router from 'vue-router'
import Home from './views/Home.vue'

Vue.use(Router)
// 路由懒加载
// route level code-splitting
// this generates a separate chunk (about.[hash].js) for this route
// which is lazy-loaded when the route is visited.
const addRouterComponent = path => import(path);

export default new Router({
  mode: 'history',
  base: process.env.BASE_URL,
  routes: [{
      path: '/',
      name: 'home',
      component: Home
    },
    {
      path: '/about',
      name: 'about',
      component: addRouterComponent('./views/About.vue')
    }
  ]
})
```

## axios 的配置
在 **scr** 目录下添加 **api** 目录，并新建两个文件：**config.js** 和 **api.js**。配置如下：


- **config.js**

```
import axios from 'axios';
import qs from 'qs';

axios.defaults.withCredentials = false;
// 请求拦截器
axios.interceptors.request.use(
  request => {
    request.data = request.data || {};
    const hasHttp = /^http(|s):\/\//.test(request.url);
    if (!hasHttp) { // 无 http or https 头处理
      request.url = `https://${request.url}`;
    }
    if (request.data) {
      request.data = {
        // commom: '', 这里可以添加一些公共的请求参数
        data: JSON.stringify(request.data)
      };
      request.data = qs.stringify(request.data);
    }
    return request;
  },
  error => Promise.reject(error)
);
// 响应拦截器
axios.interceptors.response.use(
  response => {
    const responseData = response.data;
    if (responseData.errcode !== 0) { // 错误处理
      return Promise.reject(responseData.msg || '未知错误');
    }
    responseData.result = responseData.result || {};
    return responseData;
  },
  error => Promise.reject(error)
);

export {
  axios
};
```

- **api.js**

```
import {
  axios
} from './config.js';

// 例子 1
export const aApi1Name = params => axios.post(url, params);
// 例子 2
export const aApi2Name = params => axios.get(url, params);
```

## 打包后自动部署到服务器

使用的是 **scp2**；通过向脚本传递不同参数，部署到不同的服务器路径；在根目录添加 **deploy.js** 脚本。相关配置如下：

- **package.json**

```
"buildTest": "vue-cli-service build && node deploy.js development",
"buildProd": "vue-cli-service build && node deploy.js production"
```

- **deploy.js**

```
const qs = require('qs');

// 判断是否输出帮助信息
const helpMessage = `usage: node deploy.js [deploy_target]`;
if (
  process.argv.length !== 3 ||
  process.argv[2] === '-h' ||
  process.argv[2] === '--help'
) {
  console.log(helpMessage);
  process.exit(0);
}

// 判断是否是生产环境
const isProduction = process.argv[2] === 'production';

// 设置部署的文件路径，正服 or 测服
let deployPath = '';
if (isProduction) {
  deployPath = '/production'
} else {
  deployPath = '/test'
}

// 设置服务器服务器信息
const SERVER_INFO = {
  host: '134.175.150.8',
  port: 22,
  username: '',
  password: '',
  path: deployPath
}

// 部署到服务器
client.scp('./dist/', SERVER_INFO, err => {
  if (err) {
    console.log('Deploy Failed!\n', err);
  }
})

console.log(`Deploy ${isProduction ? 'production' : 'developemt'} finished!`);
```

总结：通过上面的配置，基本可以实现 **vue** 项目从创建到配置到打包自动部署的过程。


