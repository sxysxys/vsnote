## day3-4
### VSCODE创建项目
1. 新建一个文件夹。
2. 在vscode中打开那个文件夹。
3. 点击将工作区另存为...，写上创建工作区的名称，点击保存即可创建成功。

### es6基本语法

### vue入门
- 使用`!`快捷生成html模板。
- 自定义模板。
- vue基本语法：循环、双向绑定、判断等。

### vue进阶
- 组件、全局组件
- vue路由

### axios异步请求

### element-ui
基于vue.js的组件库。

### node.js
#### node.js是什么
- node.js通俗来说相当于java的jdk，是一套js的运行环境。有了它可以直接执行js文件。
- 底层基于谷歌v8引擎，所以能够解析js文件。
- 它可以当做http服务器来使用。
- 总的来说，它就是一个运行在服务端的js环境。

#### 执行步骤
1. 下载nodejs
2. 执行js文件：`node 文件路径`进行执行。

### npm
npm:是一个js包管理依赖，类似于后端的maven。在安装node.js的时候默认就带了。

npm使用流程：
1. 在项目路径下打开终端，使用`npm init -y`建立npm的package文件(相当于maven的pom.xml)，对项目进行npm的环境初始化。
2. 设置npm的国内镜像:`npm config set registry https://registry.npm.taobao.org `
3. 查看镜像配置`npm config list`
4. 将需要的包联网下载:`npm install jquery`
5. 也可以对package文件中的包直接联网下载，拿到别人给的package文件后，直接`npm install`即可(必须在当前项目路径下执行)。

### babel
将es6转成es5，以便于浏览器的兼容性。
使用buble步骤：
1. 将项目初始化成npm包项目。
2. 在项目根路径中创建`.babelrc`文件，用来配置转码规则。
```babelrc
{
    "presets": ["es2015"],  //代表使用es2015转码器进行转码
    "plugins": []
}
```
3. 安装转码器:`npm install --save-dev babel-preset-es2015`
4. 进行转码`babel src/example.js -o dist1/compiled.js`：前面是原路径，后面是转码后的目标路径。或者转码一个目录下的所有文件`
babel src -d dist2`

### 模块化
模块化：两个js文件相互引用。
nodejs中只能执行es5的模块化引用语法，不能执行es6的模块化引用语法，需要用babel进行一个转换。
#### es5的模块化语法
1. 对需要对外开放的方法：
```javascript
module.exports={
    sum,
    sub
}
```
2. 需要引用方法的js文件中引入：
```js
const m = require('./01.js')
```
#### es6的模块化语法
1. 在对外的方法中直接写export：
```js
export function getList(){
    console.log('get list...')
}
```
2. 引用方法的时候：
```js
import {getList} from './01.js'
```
或者使用第二种方法：
1. 写export
```js
export default{
    getList(){
        console.log('getList...')
    },

    save(){
        console.log('save...')
    }
}
```
2. 引用js
```js
import m from './01.js'

m.getList()
m.save()
```
es6需要将模块化语法进行转换成es5形式才能被nodejs运行。

### webpack
作用：将多个静态文件打包成一个。在开发结束、部署之前进行的操作。
打包js文件操作步骤：
1. 安装webpack`npm install -g webpack webpack-cli`，其中-g代表全局安装，不仅可以在此demo中使用。
2. 创建js文件用于打包操作。
3. 创建一个js文件，用es5的方式将其他js文件引入这个文件。
4. 在demo的根目录下添加`webpack.config.js`文件，这个文件需要选定一个入口js文件，这个js文件中引用的所有模块以及这个js文件都会一起打包到`./dist/bundle.js`文件中：
```js
const path = require("path"); //Node.js内置模块
module.exports = {
    entry: './src/main.js', //配置入口文件
    output: {
        path: path.resolve(__dirname, './dist'), //输出路径，__dirname：当前文件所在路径
        filename: 'bundle.js' //输出文件
    }
}
```
5. 执行打包命令`webpack --mode=development/production`
其中一个开发环境，一个生产环境。开发环境打包出来的比较美观，生产环境打包出来的全在一行。

打包css文件也打包进js文件操作步骤：
1. 建立css文件。
2. 安装css加载器：`npm install --save-dev style-loader css-loader `。
3. 在`webpack.config.js`中添加上css部分：
```js
    module: {
        rules: [  
            {  
                test: /\.css$/,    //打包规则应用到以css结尾的文件上
                use: ['style-loader', 'css-loader']
            }  
        ]  
    }
```
4. 在入口js文件中引入css文件。
5. 使用`webpack --mode=development`进行打包。

### 整合vue-admin进行开发
1. 拉下vue-admin包解压放进工作区中。
2. 执行`npm install`进行包的安装。
3. 启动项目：`npm run dev`

### 对vue和前后端分离的理解
- Vue就一个90多KB的JS文件,你在页面引入 vue.min.js 就能使用Vue进行前端开发,有个IE9或更新版本的浏览器就能正常运行、
- vue.js开发需要node.js，是因为vue.js开发依赖的环境需要node.js，比如需要依赖webpack+babel构建依赖打包（本身webpack就是在node.js环境下运行的），webpack也会在开发环境中生成webpack-dev-server给开发用。vue也依赖需要依赖npm生态的各种包。开发的时候你可以把node.js当做前端工程构建工具看待。node.js本身定位就是js的runtime，可以做的事情很多，不仅仅做服务。
- 所以本质上vue项目只是用到node.js中的一些工具和功能而已，之后开发完成后需要把vue打包到一个静态服务器上面去。浏览器请求这个页面去拿到静态资源，拿到静态资源，其中的数据通过ajax去请求后端服务器。

### 创建一个vue项目
- vue-cli是一个vue的命令行工具，它基于webpack能够帮助我们快速构建一个vue项目。
[搭建项目](https://www.cnblogs.com/haitaoli/p/10304193.html)

### 使用整合的vue-admin进行前端部分的开发
- 先找到路由文件：`vue-admin-template-master/src/router/index.js`，在路由文件中添加相关的路由。
- 添加路由相关的组件component，component中写相应的页面（使用element-ui）和Vue对象中的属性。
- 写一个单独的js文件，在文件中使用它框架封装的request方法（axios），进行ajax接口请求。
- 在component中引入js文件，进行数据的渲染。

