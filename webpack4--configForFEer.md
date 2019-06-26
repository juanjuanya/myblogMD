## webpack4 — 新手教程（一）

这是一篇完整的、循序渐进的、截至发表日期**最新的**通过**webpack4配置**的线上生产的**新手友好**的干货。

本篇基于react、less、express、Antd、进行粘合。

### 资料的寻找总是艰辛又痛苦
``` 
 我也曾为webpack配置烦恼过，
 信息漫漫、教程多多、几多愁。
 调试不行，bug难找，人空瘦。
```
我真羡慕看到这篇文章的你，以下进入正文。

###  webpack配置思路初探 — 一瞄就懂

webpack是什么？

一个模块打包工具。通过分析项目结构，找到相关模块和拓展语言(Sass/Less等),将他们/打包编译成合适的格式供浏览器使用。

优点： 既然点进来了我就不说了

webpack配置的基础功能：

.1 development开发者环境下，修改前端资源文件(js/jsx、css/less、img)时，Webpack会实时重新编译打包，完成后通过热替换实时替换浏览器中资源文件。

页面无刷新的前提下(**只需ctrl + S文件，无需页面ctrl + R**)即可实时查看浏览器JS/CSS更改后的UI效果。

流程：

Ctrl + S -> Webpack recompile -> Hot replacement -> see the reault

.2 修改后端代码： 后端.js / *.json

Ctrl + s -> Server restart -> Reload page -> see the result

### 干看无用，跟着走一遍吧


![webpack配置目录](https://user-gold-cdn.xitu.io/2019/6/26/16b92195d8e9721f?w=282&h=277&f=png&s=5897)

dir | 说明
-------- | ---
fe | 前端文件夹：可放组件、JS、CSS
server | 总的html文件，在后端放着
.babelrc | Babel配置文件
.gitignore | 用来忽略node_modules
package.json | 包信息&服务端启动命令——项目配置文件
webpack.config.js | Webpack 配置文件

**命令&安装包作用**

command | 说明
-------- | ---
npm install --save-dev webpack webpack-dev-server webpack-cli webpack-dev-middleware webpack-hot-middleware | Webpack和热替换中间件
npm install -save-dev @babel/core @babel/preset-env @babel/preset-react babel-loader | Babel 插件, 主要用于转换 React 的 ES6 语法到浏览器可以识别的正常 JS 语法
cnpm install --save-dev less less-loader css-loader style-loader | Less语法编写需要用的的插件
npm install --save-dev reload | 页面主动刷新插件
npm install --save react react-dom express | react相关及框架

**各文件具体内容**

.1 server/index.html
```js
<!DOCTYPE html>
<html>

  <head>
    <title>Welcome to the NewWorld</title>
  </head>

  <body>
    <h1>Hello juanjuanya</h1> <!--请夸一下没用world的我😄 -->
    <div id="app"></div>
    <script src="./bundle.js"></script> <!-- 加载webpack编译后的bundle.js文件 -->
    <script src="/reload/reload.js"></script> <!-- js文件对应app.js中的reload(app)代码 reload插件
                                                在服务器重启后自动重加载所有包含此条的页面 -->
  </body>

</html>

```

.2.1 fe/index.js

```js
import React from 'react'
import ReactDOM from 'react-dom'
import styles from "./index.less"

const subTitle = 'React + Express 666'

ReactDOM.render(
  <div className={ styles.subtitle} >{subTitle}</div>,
  document.getElementById('app') //替换server/index中id为app的DIV区域
)

if(module.hot) {
  module.hot.accept() //当JS修改后，当前模块是热的即进行热替换，和app.js中
                     //app.use(webpackHotMiddleware(compiler)) 代码对应, 
                     //只有这两个代码都存在,
                     //JS/LESS文件修改后才会进行热替换操作
}
```
.2.2 fe/index.less

```js
.subtitle {
  color: aquamarine;
  margin: 20px;
}
```

.3 .babelrc

```js
{
  "presets": [
    "@babel/preset-env",
    "@babel/preset-react"
  ]
}

//此项配置后，不需再在package.json中配置babel选项了
```
.4 .gitignore
```js
/node_modules
```
.5 webpack.config.js
```js
const webpack = require('webpack') //导入webpack库
const path = require('path')

var hotMiddlewareScript = "webpack-hot-middleware/client?reload=true" //JS热替换失败以后重新加载页面

module.exports = {
  entry: {
    client: ["./fe", hotMiddlewareScript] //用上面的热替换规则监听fe文件夹下所有文件
  },
  mode: "development",
  module: {
    rules: [//相关代码控制js/jsx需通过babel-loader来转换React JSX文件的js语法
      {
        test:/\.(js|jsx)$/,
        include: [
          path.resolve(__dirname, 'fe')
        ],
        exclude: /node_modules/,
        use: ['babel-loader']
      },
      {
        test: /\.less$/,  //less文件自动被以下三个插件一次处理转成css样式，并保证每个CSS组件的名字有个唯一的hash值
        use: [
          {
            loader: "style-loader"
          }, 
          {
            loader: "css-loader",
            options: {
              sourceMap: true,
              modules: true,
              localIdentName: "[local]___[hash:base64:5]"
            }
          },
          {
            loader: "less-loader"
          }
        ]
      }
    ]
  },
  resolve: { //控制JS import模块时找模块https://webpack.docschina.org/configuration/resolve/
    extensions: ['*', '.js', '.jsx']
  },
  output: { //控制js/css编译后文件名和存放的目录
    path: path.resolve(__dirname, 'build'),
    publicPath: "/",
    filename: 'bundle.js'
  },
  plugins: [
    new webpack.HotModuleReplacementPlugin(),//加载插件，配合app.js,fe.index.js进行热替换
    new webpack.NoEmitOnErrorsPlugin(),//文件有语法错误时不要刷新页面，只在终端里打印错误
  ],
  devServer: {
    hot: true, //依赖于HotModuleReplacementPlugin
  }
}
```
.6 app.js
```js
var express = require('express'),
  app = express(),
  reload = require('reload'),
  rootPath = __dirname;

  //
var webpack = require("webpack"),
  webpackConfig = require("./webpack.config"),
  webpackDevMiddleware = require("webpack-dev-middleware"),
  webpackHotMiddleware = require("webpack-hot-middleware"),
  compiler = webpack(webpackConfig);

app.use(
  webpackDevMiddleware(compiler, {
    publicPath: webpackConfig.output.publicPath,
    noInfo: true,
    stats: {
      colors: true
    }
  })
);
app.use(webpackHotMiddleware(compiler));
app.use(express.static(__dirname + '/build'));

app.get('/', function (req, res) {
  res.sendFile(rootPath + '/server/index.html');
});

reload(app);

app.listen(6666, () => {
  console.log('* Server starting...');
});
```

### 极简实现华丽丽

![webpack效果显示](https://user-gold-cdn.xitu.io/2019/6/26/16b9238f9f31dae4?w=995&h=1169&f=png&s=65547)

以上就是webpack配置(最新版)的极简配置流程，配置很简单，但也足以满足新手的基础需求。 如有问题欢迎本帖下提问，我会及时解答。

webpack还有进阶版以及自动化版文档心得，敬请期待！

PS:这么狂傲的说最新版，😄，按照我这个装，当然永远是最新版，吼吼吼......

我在最开始用Webpack的时候也查阅了很多文档，但是**很少有**最新版而且全面最重要的时**能跑起来的**教程.所以在我艰难之后，希望你们都能轻松点。如有错误，欢迎指正。

我是爱分享，爱生活，爱做美食，爱运动的juanjuanya, 欢迎加入我的学习圈。之后还有更多分享，欢迎Follow me ! 


参考链接：


   
 [mldangelo/personal-site 帅哥一枚](https://github.com/mldangelo/personal-site)
      
[https://webpack.js.org/](https://webpack.js.org/)
   
[juanjuanya's github 项目还未整理，欢迎66](https://github.com/juanjuanya)


注：看了很多文档，最有用的就是这几个，这次重写配置，又配了一遍，如有相同的地方，纯属忘了之前看过哪篇了，欢迎联系，如属实就加上去。