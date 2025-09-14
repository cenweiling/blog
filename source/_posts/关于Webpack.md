---
title: 关于Webpack
date: 2024-08-13
tags: [Webpack, loader, plugin, 打包构建工具, 副作用函数, 静态分析]
categories: Webpack
---

概念：Webpack 是一种用于构建 JavaScript 应用程序的静态模块打包器，它能够以一种相对一致且开放的处理方式，加载应用中的所有资源文件（图片、CSS、视频、字体文件等），并将其合并打包成浏览器兼容的 Web 资源文件。

功能：****

+ 模块的打包：通过打包整合不同的模块文件保证各模块之间的引用和执行
+ 代码编译：通过丰富的`**loader**`可以将不同格式文件如`**.sass/.vue/.jsx**`转译为浏览器可以执行的文件
+ 扩展功能：通过社区丰富的`**plugin**`可以实现多种强大的功能，例如**代码分割、代码混淆、代码压缩、按需加载.....等等**

### 常见的loader及其作用
**<font style="color:#1DC0C9;">-babel-loader</font>**：将es6转译为es5

**<font style="color:#1DC0C9;">-file-loader</font>**：可以指定要复制和放置资源文件的位置，以及如何使用版本哈希命名以获得更好的缓存，并在代码中通过**URL**去引用输出的文件

**<font style="color:#1DC0C9;">-url-loader</font>**：和`**file-loader**`功能相似，但是可以通过指定阈值来根据文件大小使用不同的处理方式（小于阈值则返回base64格式编码并将文件的 `data-url`内联到`bundle`中）

**<font style="color:#1DC0C9;">-raw-loader</font>**：加载文件原始内容

> webpack5自身内置了`**file-loader/ url-loader/ raw-loader**`等loader，所以我们不需要再显示引入loader 只需要指定对应的type即可实现相同的功能 如`**file-loader**`等价于 `**type= "asset/resource"**`
>

**<font style="color:#1DC0C9;">-image-webpack-loader</font>**： 加载并压缩图片资源

**<font style="color:#1DC0C9;">-awesome-typescirpt-loader</font>**: 将typescript转换为javaScript 并且性能优于`**ts-loader**`

**<font style="color:#1DC0C9;">-sass-loader</font>**: 将SCSS/SASS代码转换为CSS

**<font style="color:#1DC0C9;">-css-loader</font>**: 加载CSS代码 支持模块化、压缩、文件导入等功能特性**

**<font style="color:#1DC0C9;">-style-loader</font>**: 把CSS代码注入到js中，通过`DOM` 操作去加载CSS代码

> 当我们使用类似于 `**less**` 或者 `**scss**` 等预处理器的时候，通常需要多个 **loader** 的配合使用如`**test: /\.less$/, use: ['style-loader', 'css-loader', 'less-loader']**`
>

**<font style="color:#1DC0C9;">-source-map-loader</font>**: 加载额外的`Source Map`文件

**<font style="color:#1DC0C9;">-eslint-loader</font>**: 通过ESlint 检查js代码

**<font style="color:#1DC0C9;">-cache-loader</font>**: 可以在一些开销较大的`Loader`之前添加可以将结果缓存到磁盘中，提高构建的效率

**<font style="color:#1DC0C9;">-thread-loader</font>**: 多线程打包，加快打包速度

### 常见的plugin及其作用
**<font style="color:#1DC0C9;">-splitChunkPlugin</font>**: 定义环境变量（webpack4之后可以通过指定`**mode：production/development**`实现同样效果）

**<font style="color:#1DC0C9;">-define-plugin</font>**: 定义环境变量（webpack4之后可以通过指定`**mode：production/development**`实现同样效果）

**<font style="color:#1DC0C9;">-web-webpack-plugin</font>**：为单页面应用输出HTML 性能优于`html-webpack-plugin`

**<font style="color:#1DC0C9;">-clean-webpack-plugin</font>**: 每次打包时删除上次打包的产物, 保证打包目录下的文件都是最新的

**<font style="color:#1DC0C9;">-webpack-merge</font>**： 用来合并公共配置文件,常用（例如分别配置`webpack.common.config.js/ webpack.dev.config.js/webpack.production.config.js`并将其合并）

**<font style="color:#1DC0C9;">-ignore-plugin</font>**: 忽略指定的文件，可以加快构建速度

**<font style="color:#1DC0C9;">-terser-webpack-plugin</font>**：压缩ES6的代码（tree-shaking）

**<font style="color:#1DC0C9;">-uglifyjs-webpack-plugin</font>**: 压缩js代码

**<font style="color:#1DC0C9;">-mini-css-extract-plugin</font>**: 将CSS提取为独立文件，支持按需加载x

**<font style="color:#1DC0C9;">-css-minimize-webpack-plugin</font>**：压缩CSS代码

> css文件的压缩需要`**mini-css-extract-plugin**`和`**css-minimize-webpack-plugin** `的配合使用 即先使用`**mini-css-extract-plugin**`将css代码抽离成单独文件，之后使用` **css-minimize-webpack-plugin**`对css代码进行压缩
>

**<font style="color:#1DC0C9;">-serviceworker-webpack-plugin</font>**: 为离线应用增加离线缓存功能

**<font style="color:#1DC0C9;">-ModuleconcatenationPlugin</font>**: 开启`**Scope Hositing**` 用于合并提升作用域， 减小代码体积

**<font style="color:#1DC0C9;">-copy-webpack-plugin</font>**： 在构建的时候，复制静态资源到打包目录。

**<font style="color:#1DC0C9;">-compression-webpack-plugin</font>**: 生产环境采用`**gzip**`压缩JS和CSS

**<font style="color:#1DC0C9;">-ParalleUglifyPlugin</font>**： 多进程并行压缩js

**<font style="color:#1DC0C9;">-webpack-bundle-analyzer</font>**: 可视化webpack输出文件大小的根据

**<font style="color:#1DC0C9;">-speed-measure-webpack-plugin</font>**: 用于分析各个loader和plugin的耗时，可用于性能分析

**<font style="color:#1DC0C9;">-webpack-dashboard</font>**: 可以更友好地展示打包相关信息

**<font style="color:#1DC0C9;">-Webpack Analysis</font>**：webpack 官方提供的可视化分析工具

**<font style="color:#1DC0C9;">-BundleAnalyzerPlugin</font>**：性能分析插件，可以在运行后查看是否包含重复模块/不必要模块等

### loader和plugin的区别
**<font style="color:#1DC0C9;">Loader：</font>**

Loader本质上是一个函数，负责代码的转译，即对接收到的内容进行转换后将转换后的结果返回 ，因为 Webpack 只认识 JavaScript，所以 Loader 就成了翻译官，对其他类型的资源进行转译的预处理工作。

配置Loader通过在 **module.rules **中以数组的形式配置，每一项都是一个 Object，内部包含了 test(类型文件)、loader、options (参数)等属性。

**<font style="color:#1DC0C9;">Plugin：</font>**

Plugin可以扩展 Webpack 的功能，本质上是一个带有 **apply(compiler)** 的函数，基于 [tapable](https://link.juejin.cn/?target=https%3A%2F%2Fwww.npmjs.com%2Fpackage%2Ftapable) 这个事件流框架来监听**webpack**构建/打包过程中发布的hooks来通过自定义的逻辑和功能来改变输出结果。 

Plugin通过 **plugins** 以数组的形式配置，每一项是一个 Plugin 的实例，参数都通过构造函数传入。

**总结：**

**Loader **主要负责将代码转译为**webpack**可以处理的JavaScript代码，而 **Plugin** 更多的是负责通过接入**webpack**构建过程来影响构建过程以及产物的输出，**Loader**的职责相对比较**单一**简单，而 **Plugin **更为丰富多样。

### loader执行顺序
##### 分类
    - **<font style="color:#1DC0C9;">pre</font>**：前置loader 用法：**module.rules** 中 **enforce** 属性指定
    - **<font style="color:#1DC0C9;">normal</font>**：普通loader 用法：**module.rules** 中 **enforce** 属性指定
    - **<font style="color:#1DC0C9;">inline</font>**：内联loader（不建议使用）用法：在import语句中显示指定loader 举例：`import Styles from 'style-loader|css-loader?modules!./styles.css';` 含义：用 css-loader 和 style-loader 处理 styles.css 文件，通过 ! 将资源中的loader分开
    - **<font style="color:#1DC0C9;">post</font>**：后置loader 用法：**module.rules** 中 **enforce** 属性指定

##### 执行顺序
    - 4类loader的执行顺序：`pre > normal > inline > post`
    - 相同优先级的loader执行顺序：`从右到左，从下到上`

##### pitching loader
    - loader 上的 **<font style="color:#1DC0C9;">pitch</font>** 方法

```javascript
module.exports = function (content) {
  return content;
};
module.exports.pitch = function (remainingRequest, precedingRequest, data) {
  console.log("do somethings");
};
```

    - webpack 会先从左到右执行 loader 链中的每个 loader 上的 pitch 方法（如果有），然后再从右到左执行 loader 链中的每个 loader 上的普通 loader 方法

![pitching_loader_1](images/pitching_loader_1.png)

    - 在这个过程中如果任何 pitch 有返回值，则 loader 链被阻断。webpack 会跳过后面所有的的 pitch 和 loader，直接进入上一个 loader 

![pitching_loader_2](images/pitching_loader_2.png)

### webpack构建流程
Webpack 的运行流程是一个串行的过程，从启动到结束会依次执行以下流程：

+ **<font style="color:#1DC0C9;">初始化参数</font>**：从配置文件和 Shell 语句中读取与合并参数，得出最终的参数
+ **<font style="color:#1DC0C9;">开始编译</font>**：用上一步得到的参数初始化 Compiler 对象，加载所有配置的插件，执行对象的 run 方法开始执行编译
+ **<font style="color:#1DC0C9;">确定入口</font>**：根据配置中的 entry 找出所有的入口文件
+ **<font style="color:#1DC0C9;">编译模块</font>**：从入口文件出发，调用所有配置的 Loader 对模块进行翻译，再找出该模块依赖的模块，再递归本步骤直到所有入口依赖的文件都经过了本步骤的处理
+ **<font style="color:#1DC0C9;">完成模块编译</font>**：在经过第4步使用 Loader 翻译完所有模块后，得到了每个模块被翻译后的最终内容以及它们之间的依赖关系
+ **<font style="color:#1DC0C9;">输出资源</font>**：根据入口和模块之间的依赖关系，组装成一个个包含多个模块的 Chunk，再把每个 Chunk 转换成一个单独的文件加入到输出列表，这步是可以修改输出内容的最后机会
+ **<font style="color:#1DC0C9;">输出完成</font>**：在确定好输出内容后，根据配置确定输出的路径和文件名，把文件内容写入到文件系统

在以上过程中，`Webpack` 会在特定的时间点广播出特定的事件，插件在监听到感兴趣的事件后会执行特定的逻辑，并且插件可以调用 Webpack 提供的 API 改变 Webpack 的运行结果。

简单说

+ **<font style="color:#1DC0C9;">初始化</font>**：启动构建，读取与合并配置参数，加载 Plugin，实例化 Compiler
+ **<font style="color:#1DC0C9;">编译</font>**：从 Entry 出发，针对每个 Module 串行调用对应的 Loader 去翻译文件的内容，再找到该 Module 依赖的 Module，递归地进行编译处理
+ **<font style="color:#1DC0C9;">输出</font>**：将编译后的 Module 组合成 Chunk，将 Chunk 转换成文件，输出到文件系统中

### 文件指纹
概念：文件指纹是打包后输出的文件名的后缀

##### 种类：
+ **<font style="background-color:rgba(222,253,255,1);"> Hash </font>**：和整个项目的构建相关，只要项目文件有修改（**<font style="background-color:rgba(222,253,255,1);"> compilation </font>**实例改变），整个项目构建的 Hash 值就会更改
+ **<font style="background-color:rgba(222,253,255,1);"> Chunkhash </font>**：和 Webpack 打包的 chunk 有关，不同的 entry 会生出不同的 chunkhash，（不同 Chunkhash 之间的变化互不影响）
+ <font style="background-color:rgba(222,253,255,1);"> </font>**<font style="background-color:rgba(222,253,255,1);">Contenthash</font>**<font style="background-color:rgba(222,253,255,1);"> </font>：根据文件内容来定义 hash，文件内容不变，则 Contenthash 不变

##### 使用：
+ JS文件：使用 **<font style="background-color:rgba(222,253,255,1);"> Chunkhash </font>**
+ CSS文件：使用 <font style="background-color:rgba(222,253,255,1);"> </font>**<font style="background-color:rgba(222,253,255,1);">Contenthash</font>**<font style="background-color:rgba(222,253,255,1);"> </font>
+ 图片等静态资源： 使用 **<font style="background-color:rgba(222,253,255,1);"> Hash </font>**

> 生产环境的output为了区分版本变动，通过**<font style="background-color:rgba(222,253,255,1);"> Contenthash </font>**来达到清理缓存及时更新的效果，而开发环境中为了加快构建效率，一般不引入**<font style="background-color:rgba(222,253,255,1);"> Contenthash </font>**
>

##### JS的文件指纹设置
设置 output 的 filename，用 chunkhash

```javascript
module.exports = {   
  entry: {        
    app: './scr/app.js',        
    search: './src/search.js'    
  },    
  output: {        
    filename: '[name][chunkhash:8].js', // chunkhash:8 保留8位    
    path:__dirname + '/dist'    
  }
}

```

##### CSS的文件指纹设置
设置 MiniCssExtractPlugin 的 filename，用 contenthash

```javascript
module.exports = {    
  entry: {        
    app: './scr/app.js',        
    search: './src/search.js'   
  },    
  output: {        
    filename: '[name][chunkhash:8].js',        
    path:__dirname + '/dist'   
  },    
  plugins:[        
    new MiniCssExtractPlugin({            
      filename: `[name][contenthash:8].css`        })   
  ]
}

```















##### 图片的文件指纹设置
设置 file-loader 的 filename，用 hash，**webpack5后内置file-loader/url-loader/raw-loader，配置assets属性即可，无需再引入loader**

占位符名称及含义

+ ext 资源后缀名
+ name 文件名称
+ path 文件的相对路径
+ folder 文件所在的文件夹
+ contenthash 文件的内容hash，默认是md5生成
+ hash 文件内容的hash，默认是md5生成
+ emoji 一个随机的指代文件内容的emoj

```javascript
const path = require('path');
module.exports = {    
  entry: './src/index.js',    
  output: {        
    filename:'bundle.js',        
    path:path.resolve(__dirname, 'dist')   
  },    
  module:{        
    rules:[{            
      test:/\.(png|svg|jpg|gif)$/,            
      use:[{                
        loader:'file-loader',                
        options:{                    
          name:'img/[name][hash:8].[ext]'               
        }           
      }]       
    }]   
  }
}

// webpack5配置参考如下：
...
// rules:[
//   {
//         test: /\.(png|jpe?g|gif|webp|svg)$/,
//         type: 'asset',
//         parser: {
//           dataUrlCondition: {
//             // 小于10kb的图片转base64
//             // 优点：减少请求数量  缺点：体积会更大
//             maxSize: 10 * 1024 // 10kb 
//           }
//         },
//         generator: {
//           // 输出图片名称
//           // [hash:10] hash值取前10位
//           filename: 'static/images/[hash:10][ext][query]'
//         }
//       },
//   {
//         test: /\.(ttf|woff2?|mp3|mp4|avi)$/,
//         type: 'asset/resource',
//         generator: {
//           // 输出名称
//           // [hash:10] hash值取前10位
//           filename: 'static/media/[hash:10][ext][query]'
//         }
//       },
// ]
// ...

```

### Babel的原理
概述：大多数JavaScript Parser遵循 **estree** 规范，Babel 最初基于 **acorn** 项目(轻量级现代 JavaScript 解析器) 。

**babel** 可以将代码转译为想要的目标代码，并且对目标环境不支持的**api** 自动 **<font style="background-color:rgba(222,253,255,1);"> polyfill </font>** 。而 **babel **实现这些功能的流程是 **<font style="background-color:rgba(222,253,255,1);"> 解析（parse）-转换（transfrom）-生成（generator）</font>** 

+ **<font style="background-color:rgba(77, 208, 225, 0.08);">解析</font>**：根据代码生成对应的 **AST** 结构
    - 词法分析：将代码(字符串)分割为 **token** 流，即语法单元成的数组
    - 语法分析：分析 **token **流(上面生成的数组)并生成 AST
+ **<font style="background-color:rgba(77, 208, 225, 0.08);">转换</font>**：遍历 **AST** 节点并生成新的 **AST** 节点
+ **<font style="background-color:rgba(77, 208, 225, 0.08);">生成</font>**：根据新的 **AST** 生成目标代码

### 文件监听原理
在发现源码发生变化时，自动重新构建出新的输出文件

Webpack开启监听模式，有两种方式：

+ 启动 webpack 命令时，带上 --watch 参数
+ 在配置 webpack.config.js 中设置 watch:true

缺点：每次需要手动刷新浏览器

原理：轮询判断文件的最后编辑时间是否变化，如果某个文件发生了变化，并不会立刻告诉监听者，而是先缓存起来，等 **aggregateTimeout** 后再统一执行。

```javascript
watch: true, // 默认false，也就是不开启，只有开启监听模式时，watchOptions才有意义
watchOptions: {
    // 不监听的文件或者文件夹，忽略一些大型的不经常变化的文件可以提高构建速度，支持正则匹配
    ignored: /node_modules/,
    //监听到变化会等多少时间再执行，默认300ms
    aggregateTimeout: 300,
    //判断文件是否发生变化是通过不断轮询指定文件有没有变化实现的，默认每秒问1000次
    poll: 1000
}

```



### 热更新原理
参考链接：[轻松理解webpack热更新原理](https://juejin.cn/post/6844904008432222215?searchId=20250725014743FA245E0AE4142AA9178D) [从零实现webpack热更新HMR](https://juejin.cn/post/6844904020528594957?searchId=20250725014743FA245E0AE4142AA9178D)

视频讲解：[Webpack 热更新原理与实战](https://www.bilibili.com/video/BV1To4y1f7Wo?spm_id_from=333.788.player.player_end_recommend&vd_source=a63e68ff2d1fc881d5dfb3ccab9ccf89)

概述：模块热替换（Hot Module Replacement，HMR）是 webpack 提供的最有用的功能之一。当我们**对代码修改并保存后，webpack将会对代码进行新的打包，并将新的模块发送到浏览器端，浏览器用新的模块替换掉旧的模块**，以实现在不刷新浏览器的前提下更新页面。

刷新我们一般分为两种：

+ 一种是页面刷新，不保留页面状态，就是简单粗暴，直接 **<font style="background-color:#FBE4E7;">window.reload()</font>**
+ 另一种是基于 **<font style="background-color:#FBE4E7;">WDS（Wepack-dev-server）</font>**的模块热替换，只需要局部刷新页面上发生变化的模块，同时可以保留当前的页面状态，比如复选框的选中状态、输入框的输入等。

##### 整体工作流程
1. 文件修改：开发者保存修改后的文件
2. Webpack 重新编译：生成新的模块代码和补丁（patch）
3. 通知客户端：通过 websocket 向浏览器发送更新消息
4. 客户端应用更新：获取新模块并替换旧模块
5. 模块热替换：仅更新变化的模块，保持应用状态



![pic_HMR](images/pic_HMR.png)

详细解析图如下（含源码指导版）：

![热更新](images/热更新.png)





##### 核心组成部分
###### 2.1 Webpack-dev-server (或 webpack-dev-middleware)
+ 启动一个 Express 服务器
+ 创建 websocket 服务（默认端口 8080）
+ 监听文件变化并触发重新编译
+ 通过 _webpack_hmr 端点与客户端通信

###### 2.2 HMR Runtime（客户端部分）
注入到 bundle 中的代码，负责：

+ 建立与 dev-server 的 websocket 连接
+ 接收更新通知
+ 发起 JSONP 请求获取更新模块
+ 执行模块替换逻辑

###### 2.3 HMR Plugin（服务端部分）
Webpack 内置的 HotModuleReplacementPlugin 负责：

+ 生成每个模块的 HMR 标识（hash）
+ 生成 manifest 文件（记录模块更新信息）
+ 生成 update 补丁文件（[oldHash].hot-update.json/.js）

##### 模块热更新策略
###### 3.1不同类型模块的处理
1. 普通 JS 模块：
    - 直接替换模块导出对象
    - 需要手动编写 module.hot.accept 逻辑
2. 样式模块（通过 style-loader）：
    - 自动处理：新样式替换旧样式（无需页面刷新）
    - 实现原理：style-loader 会注入特殊的 HMR 代码
3. React/Vue 组件：
    - 需要框架特定的 HMR 支持
    - React: 使用 react-hot-loader 或 React Refresh
    - Vue: vue-loader 内置支持

###### 3.2更新传播机制
+ 接受依赖：当模块 A 调用 module.hot.accept，表示它能够接受自身或依赖模块的更新
+ 冒泡更新：如果父模块不接受子模块更新，更新会向上冒泡直到找到接受者
+ 无接受者：如果没有模块接受更新，则 fallback 到整页刷新

### 代码分割
参考文章：[webpack优化之玩转代码分割和公共代码提取](https://juejin.cn/post/6844904001792655373?searchId=202507231703353AC77072FBD359203430)

概述：代码分割是 Webpack 的核心功能之一，它允许你将代码拆分成多个 bundle，然后可以按需加载或并行加载，从而优化应用性能。

##### 为什么需要代码分割
1. 减小初始加载体积：将应用拆分成多个 bundle，用户只需加载当前需要的代码
2. 提高加载速度：并行加载多个 bundle 比加载单个大文件更快
3. 缓存优化：将不常变动的代码单独打包，利用浏览器缓存

##### 代码分割的三种主要方式
1. 入口起点(Entry Points)

```javascript
module.exports = {
  entry: {
    app: './src/app.js',
    vendor: './src/vendor.js'
  },
  output: {
    filename: '[name].bundle.js'
  }
}
```

缺点：如果多个入口共享模块，这些模块会被重复打包到各个 bundle 中(用SplitChunksPlugin解决)

2. 防止重复(SplitChunksPlugin)

Webpack 4+ 内置了 SplitChunksPlugin，可以自动拆分公共依赖：

```javascript
module.exports = {
  optimization: {
    splitChunks: {
      chunks: 'all',
      minSize: 30000, // 模块的最小体积
      minChunks: 1, // 模块的最小被引用次数
      cacheGroups: {
        vendors: {
          test: /[\\/]node_modules[\\/]/,
          priority: -10
        },
        default: {
          minChunks: 2,
          priority: -20,
          reuseExistingChunk: true
        }
      }
    }
  }
};
```

3. 动态导入(Dynamic Imports)

使用 ES6 的 import() 语法或 Webpack 特定的 require.ensure：

1. 默认行为：单独打包

当使用 import() 动态导入语法时：

```javascript
// 点击按钮时动态加载模块
button.addEventListener('click', () => {
  import('./math.js').then(math => {
    console.log(math.add(16, 26));
  });
});
```

    - Webpack 会将 math.js 及其依赖打包成一个独立的 chunk（如 1.bundle.js）。
    - 运行时按需加载：只有当代码执行到 import() 时，浏览器才会发起网络请求加载这个 chunk。
2. 控制打包名称（魔法注释）

通过 webpackChunkName 注释可以自定义 chunk 名称：

```javascript
import(/* webpackChunkName: "math-lib" */ './math.js')
```

    - 生成的文件名会变成 math-lib.bundle.js 而非数字 ID。
    - 适合给重要模块命名，便于调试和长期缓存。
3. 依赖关系处理
    - 如果动态导入的模块 依赖其他模块，这些依赖会被 一起打包到同一个 	chunk 中。
    - 如果多个动态导入的模块 共享依赖，Webpack 默认会 重复打包（除非通过 SplitChunksPlugin 优化）。

预获取/预加载模块

Webpack 4.6+ 支持使用魔法注释实现资源预加载：

```javascript
import(/* webpackPrefetch: true */ './path/to/LoginModal.js');
import(/* webpackPreload: true */ './path/to/ChartComponent.js');
```

+ prefetch：浏览器空闲时加载，用于未来可能需要的资源
+ preload：与父 chunk 并行加载，用于当前导航可能需要的资源

##### 代码分割最佳实践
+ 按路由分割：为每个路由创建单独的 chunk
+ 提取公共依赖：将第三方库(vendor)和公共模块单独打包
+ 合理使用动态导入：对非关键功能使用懒加载
+ 监控 bundle 大小：使用 webpack-bundle-analyzer 分析包内容

示例配置：

```javascript
module.exports = {
  // ...
  optimization: {
    splitChunks: {
      chunks: 'all',
      maxSize: 244 * 1024, // 尝试将大于244KB的块拆分成更小的块
      cacheGroups: {
        reactVendor: {
          test: /[\\/]node_modules[\\/](react|react-dom)[\\/]/,
          name: 'react-vendor',
          chunks: 'all',
        },
        utilityVendor: {
          test: /[\\/]node_modules[\\/](lodash|moment|axios)[\\/]/,
          name: 'utility-vendor',
          chunks: 'all',
        },
      },
    },
  },
};
```

### 手写loaders和plugin
文档：文稿/learning/chl-learning/webpack5/kejian/course/webpack_docs

代码：文稿/learning/chl-learning/webpack5/SOURCE

学习视频：[尚硅谷Webpack5入门到原理（面试开发一条龙](https://www.bilibili.com/video/BV14T4y1z7sw/?spm_id_from=333.788.videopod.episodes&vd_source=a63e68ff2d1fc881d5dfb3ccab9ccf89&p=42)

### tree-shaking
参考链接：[Webpack 原理系列九：Tree-Shaking 实现原理](https://juejin.cn/post/7002410645316436004?searchId=2025080400284774D9A2A868649273E08D)

概述：前端中的tree-shaking可以理解为通过工具"摇"我们的JS文件，将其中用不到的代码"摇"掉，是一个性能优化的范畴。具体来说，在 webpack 项目中，有一个入口文件，相当于一棵树的主干，入口文件有很多依赖的模块，相当于树枝。实际情况中，虽然依赖了某个模块，但其实只使用其中的某些功能。通过 tree-shaking，将没有使用的模块摇掉，这样来达到删除无用代码的目的。

支持tree-shaking的构建工具：

+ Rollup（最早实现）
+ Webpack
+ Closure compiler（google的）

#### 工作原理
##### 依赖关系图分析
打包工具会构建模块的依赖关系图，标记哪些导出被实际使用

##### 静态分析阶段
1. 识别导出：分析每个模块的 export 语句。
2. 追踪引用：从入口文件开始，追踪所有 import 的依赖链。
3. 标记失效代码：未被引用的导出标记为“可删除”。

##### 删除阶段
通过压缩工具（如 Terser）移除标记为未使用的代码。

#### 实现条件
##### 1.必须使用ES Module（ESM）
+ Tree Shaking 依赖 ESM 的 静态结构（import/export 必须在顶层声明）。
+ CommonJS 无法被优化（require 是动态的，无法在编译时分析）。

```javascript
// ✅ 可被 Tree Shaking（ESM）
import { usedFunc } from './utils';
usedFunc();

// ❌ 无法被 Tree Shaking（CommonJS）
const utils = require('./utils');
utils.usedFunc();
```



##### 2.避免副作用代码
+ 工具会假设所有代码可能有副作用（如修改全局变量），除非显式声明。
+ 通过 package.json 的 sideEffects 字段标记无副作用的模块：

```javascript
// package.json
{
  "sideEffects": false,  // 整个包无副作用
  "sideEffects": ["*.css"]  // 仅 CSS 文件有副作用
}
```

#### 副作用代码（Side Effects）
概述**：**副作用代码是指在执行时会对外部环境产生可观察影响的代码，这类代码不仅返回计算结果，还会修改外部状态或与系统交互。理解副作用对编写可维护、可优化的代码至关重要。

##### 核心概念
###### 定义
+ 纯函数（无副作用）：输出仅由输入决定，不修改外部状态（如数学函数 Math.sqrt(4)）。
+ 副作用代码：执行时会产生额外影响，例如：
    - 修改全局变量
    - 操作 DOM
    - 发起网络请求
    - 读写文件/数据库
    - 打印日志

###### 示例对比
| **无副作用** | **有副作用** |
| --- | --- |
| `const sum = (a, b) => a + b` | `let count = 0;`   `const add = () => { count++ }` |
| `function capitalize(str) { return str.toUpperCase() }` | `document.title = "新标题"` |




##### 副作用代码的类型
###### 显式副作用
直接对外部环境产生影响的代码：

```javascript
// 修改全局变量
window.user = { name: "Alice" };

// 操作 DOM
document.body.style.backgroundColor = "red";

// 写入文件（Node.js）
fs.writeFileSync("log.txt", "数据已更新");
```

###### 隐式副作用
通过依赖外部状态间接产生副作用：

```javascript
// 依赖外部变量（结果不可预测）
let base = 10;
const impureAdd = (x) => base + x;

// 读取用户输入（外部依赖）
const input = prompt("请输入内容");
```



##### 副作用的影响
###### 对代码优化的阻碍
+ Tree Shaking失效：打包工具无法安全删除未使用的副作用代码：

```javascript
// 即使未使用，以下代码也会被保留
Array.prototype.customMethod = () => {}; // 修改原型链
```

+ 难以测试和调试：依赖外部状态的代码行为不可预测。

###### 对函数式编程的挑战
副作用违背引用透明性（同一输入始终返回同一输出），例如：

```javascript
// ❌ 非引用透明（依赖 Date 外部状态）
const getTime = () => new Date().toISOString();
```



##### 如何管理副作用
###### 隔离副作用
将副作用代码集中管理，与纯逻辑分离：

```javascript
// 纯函数（核心逻辑）
function calculateTotal(items) {
  return items.reduce((sum, item) => sum + item.price, 0);
}

// 副作用集中处理
function renderTotal(total) {
  document.getElementById("total").textContent = `$${total}`;
}

// 组合使用
const items = [{ price: 10 }, { price: 20 }];
renderTotal(calculateTotal(items));
```

###### 显式标记副作用
通过 `package.json` 声明模块的副作用，帮助打包工具优化：

```javascript
{
  "sideEffects": [
    "**/*.css",  // CSS 文件有副作用（注入样式）
    "src/polyfills.js" //  polyfill 修改全局对象
  ]
}
```

###### 使用函数式编程技术
+ 不可变数据：避免直接修改对象/数组（使用 `...` 或 `Object.assign`）。
+ 副作用延迟执行：如 React 的 `useEffect`、Redux 的中间件。

#### 静态分析（Static Analysis）
静态分析是指在不实际运行代码的情况下，通过分析源代码或编译后的中间表示（如 AST）来检查代码的结构、语法、依赖关系以及潜在问题的一种技术。它在编程语言、编译器、安全分析和开发工具中广泛应用。

##### 静态分析的核心概念
###### 1.1 静态分析 vs 动态分析
| **特性****** | **静态分析** | **动态分析** |
| --- | --- | --- |
| **执行时机** | 代码运行前（编译时/构建时） | 代码运行时 |
| **输入数据** | 源代码、AST、字节码等静态结构 | 程序运行时的内存、I/O、状态等 |
| **典型工具** | ESLint、TypeScript、Flow、SonarQube | Chrome DevTools、Valgrind、Fuzz 测试 |
| **优势** | 提前发现问题，不影响运行时性能 | 捕捉运行时行为（如内存泄漏） |
| **局限性** | 无法分析动态行为（如 `eval`） | 需要实际运行，可能漏检某些路径 |




###### 1.2 静态分析的主要目标
+ 代码质量检查（如未使用的变量、代码风格）
+ 安全漏洞检测（如 SQL 注入、XSS）
+ 性能优化（如 Dead Code Elimination）
+ 依赖分析（如 Tree Shaking）
+ 类型检查（如 TypeScript）



##### 静态分析的关键技术
###### 2.1 抽象语法树（AST）
静态分析的核心数据结构，将代码转换为树状结构，便于工具分析

```javascript
// 示例代码
const sum = (a, b) => a + b;

// 对应的AST（简化版）
{
  type: "Program",
    body: [{
    type: "VariableDeclaration",
    declarations: [{
      type: "VariableDeclarator",
      id: { type: "Identifier", name: "sum" },
      init: {
        type: "ArrowFunctionExpression",
        params: [
          { type: "Identifier", name: "a" },
          { type: "Identifier", name: "b" }
        ],
        body: {
          type: "BinaryExpression",
          operator: "+",
          left: { type: "Identifier", name: "a" },
          right: { type: "Identifier", name: "b" }
        }
      }
    }]
  }]
}
```

工具：Babel、ESlint、Prettier均基于AST操作代码



###### 2.2 数据流分析（Data Flow Analysis）
跟踪变量和值的流动，用于检测：

+ 未初始化的变量
+ 不可达代码
+ 常量传播优化


###### 2.3 控制流分析（Control Flow Analysis）
+ 检测无限循环
+ 识别异常处理遗漏

```javascript
function risky(x) {
  if (x > 0) {
    return "ok";
  }
  // 静态分析可警告：缺少 else 分支，可能返回 undefined
}
```

###### 2.4 类型检查（Type Checking）
在编译时验证类型一致性，如TypeScript的静态类型系统：

```javascript
function greet(name: string) {
  return `Hello, ${name}`;
}
greet(42); // 静态分析报错：类型不匹配
```

