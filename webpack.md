###  四个核心概念

    * Entry：入口起点(entry point)指示 webpack 应该使用哪个模块，来作为构建其内部依赖图的开始。
    * Output：output 属性告诉 webpack 在哪里输出它所创建的 bundles，以及如何命名这些文件，默认值为 ./dist。
    * Loader：loader 让 webpack 能够去处理那些非 JavaScript 文件（webpack 自身只能解析： JavaScript、json）。
    * Plugins：插件则可以用于执行范围更广的任务。插件的范围包括，从打包优化和压缩，一直到重新定义环境中的变量等。
### webpack基本配置

````js
const { resolve } = require('path');

module.exports = {
  entry: './src/js/index.js',
  output: {
    filename: 'js/build.js',
    path: resolve(__dirname, 'build')
  },
  module: {
    rules: []
  },
  plugins: [],
  mode: 'development' // 开发模式
  // mode: 'production'   // 生产模式
};
````

### 样式文件

````js
stytle-loader	css-loader	less-loader
mini-css-extract-plugin		// 抽取css
optimize-css-assets-webpack-plugin		// 压缩css

npm i stytle-loader css-loader less-loader mini-css-extract-plugin optimize-css-assets-webpack-plugin -D

const MiniCssExtractPlugin = require('mini-css-extract-plugin');
const OptimizeCssAssetsWebpackPlugi = require('optimize-css-assets-webpack-plugin');

// css 兼容  postcss --> postcss-loader   postcss-preset-env   package.json中定义兼容浏览器 browserlist   默认查看production生产环境  修改development   process.env.MODE_ENV = 'development';

package.json文件加入:
"browserslist": {
  "development": [
    "last 1 chrome version",
    "last 1 firefox version",
    "last 1 safari version",
    "last 1 opera version"
  ],
  "production": [
    ">0.2%",
    "not dead",
    "not op_mini all"
  ]
}

  module: {
    rules: [
      {
        test: /\.css$/,
        use: [MiniCssExtractPlugin.loader, 'css-loader']
      },
      {
        test: /\.less$/,
        use: [
          {
            loader: MiniCssExtractPlugin.loader,
            options: {
              // 提取css文件可能导致图片路径出错
              publicPath: '../'
            }
          },
          'css-loader',
          {
            loader: 'postcss-loader',
            options: {
              ident: 'postcss',
              plugins: () => [require('postcss-preset-env')()]
            }
          },
          'less-loader'
        ]
      }
    ]
  }
  
  plugins: [
    new MiniCssExtractPlugin({
      filename: 'css/main.css'
    }),
    new OptimizeCssAssetsWebpackPlugi()
  ],
````

### 图片

````js
 npm i url-loader  file-loader -D
 
{
  test: /\.(jpg|png|gif)$/,
  loader: 'url-loader',
  options: {
    // 图片小于8kb base64处理图片
    limit: 8 * 1024,
    // [name]原图片名字 [hash:10]哈希值前10位 [ext]源文件扩展名
    name: '[name]-[hash:10].[ext]',
    // 存放图片文件夹
    outputPath: 'image',
	// 使用commonjs 兼容html-loader处理图片
    esModule: false,
  }
}
````

### html文件使用图片

````js
npm  i html-loader -D

// 问题: html-loader处理图片引入的是commonjs  url-loader默认使用es6模块解析  关闭url-loader的es6模块化  使用		   commonjs解析

{
  test: /\.html$/,
  loader: 'html-loader'
}
````

### 其他文件

````js
npm i file-loader -D

{
  exclude: /\.(js|html|css|jpg|png|gif)$/,
  loader: 'file-loader',
  options: {
    outputPath: 'media'
  }
}
````

### 打包html模板

```js
npm i html-webpack-plugin -D

new HtmlWebpackPlugin({
  template: './src/index.html',
  // 压缩html
  minify: {
    // 删除空格
    collapseWhitespace: true,
    // 移除注释
    removeComments: true
  }
})
```

### clean-webpack-plugin

```js
自动清除打包输出的目录

npm i clean-webpack-plugin -D

const { CleanWebpackPlugin } = require('clean-webpack-plugin');
plugin:[
  // 现在的版本不用指定文件路径了，直接调用new CleanWebpackPlugin()
  new CleanWebpackPlugin()
]
```



### eslint 语法检查

```js
npm i eslint-loader eslint eslint-config-airbnb-base  eslint-plugin-import -D

//设置检查规则:	package.json中eslintConfig中配置  "eslintConfig": {"extends": "airbnb-base"}

{
  test: /\.js$/,
  loader: 'eslint-loader',
  exclude: 'node-moudles',
  // 一个js文件被多个loader执行  设置优先执行
  enforce: 'pre',
  options: {
    // 自动修复
    fix: true
  }
}
```

### js兼容

```js
>1. 基本js兼容性处理 --> babel-loader  @babel/preset-env @babel/core
      问题: 只能转换基本语法
 2. 全部js兼容  --> @babel/polyfill 秩只需在js文件中引入
      问题: 全部引入体积过大
>3. 按需加载兼容处理: core-js 
 
 npm i babel-loader  @babel/preset-env @babel/core core-js -D

{
  test: /\.js$/,
  exclude: 'node-moudle',
  loader: 'babe-loader',
  options: {
    //  预设: babel做怎样兼容
    presets: [
      [
        '@babel/preset-env',
        {
          // 按需加载
          useBuiltIns: 'usage',
          // 指定 corejs版本
          corejs: {
            version: 3
          },
          // 指定兼容浏览器版本
          targets: {
            chrome: '60',
            firefox: '50',
            ie: '10',
            safari: '10',
            edge: '17'
          }
        }
      ]
    ]
  }
}
```

### devServer

````js
npm i webpack-dev-server -D

启动服务器	npx webpack-dev-server

devServer: {
  // 项目构建后路径
  contentBase: resolve(__dirname, 'build'),
  // 启动gzip压缩
  compress: true,
  // 端口
  port: 8000,
  // 自动打开浏览器
  open: true
}
````

### HMR

```js
HMR: hot moudle replaceement 热模替换

样式文件: 支持HMR,因为style-loader内部实现了

html文件: 默认不支持HMR,同时会导致问题 html文件不能热更新了
	解决: 修改entry入口 将html文件按引入, entry: ['./src/js/index.js', './src/index.html']

js: 默认不能使用HMR功能（修改一个 js 模块所有 js 模块都会刷新）--> 实现 HMR 需要修改 js 代码（添加支持 HMR 功能的代码）
	注意: HMR对于js 只能热更新非入口js文件
	if (module.hot) {
      // 一旦 module.hot 为true，说明开启了HMR功能。 --> 让HMR功能代码生效
      module.hot.accept('./kai.js', function() {
        // 方法会监听 print.js 文件的变化，一旦发生变化，只有这个模块会重新打包构建，其他模块不会。
        // 会执行后面的回调函数
        increment();
      });
    }

// 凯琪热模替换
devServer:{
  hot:true
}
```

### source-map

```js
提供源代码到构建后代码的映射的技术 （如果构建后代码出错了，通过映射可以追踪源代码错误）

devtool: 'eval-source-map'

参数：[inline-|hidden-|eval-][nosources-][cheap-[module-]]source-map
1. source-map：外部，错误代码准确信息 和 源代码的错误位置
2. inline-source-map：内联，只生成一个内联 source-map，错误代码准确信息 和 源代码的错误位置                           3. hidden-source-map：外部，错误代码错误原因，但是没有错误位置（为了隐藏源代码），不能追踪源代码错误，只能提示到构建后代码的错误位置
4. eval-source-map：内联，每一个文件都生成对应的 source-map，都在 eval 中，错误代码准确信息 和 源代码的错误位
5. nosources-source-map：外部，错误代码准确信息，但是没有任何源代码信息（为了隐藏源代码）
6. cheap-source-map：外部，错误代码准确信息 和 源代码的错误位置，只能把错误精确到整行，忽略列
7. cheap-module-source-map：外部，错误代码准确信息 和 源代码的错误位置，module 会加入 loader 的 source-map
                                       
内联 和 外部的区别：1. 外部生成了文件，内联没有 2. 内联构建速度更快
                                       
开发/生产环境可做的选择：
开发环境：需要考虑速度快，调试更友好

速度快( eval > inline > cheap >... )

    eval-cheap-souce-map

    eval-source-map

    调试更友好

    souce-map

    cheap-module-souce-map

    cheap-souce-map

    最终得出最好的两种方案 --> eval-source-map（完整度高，内联速度快） / eval-cheap-module-souce-map（错误提示忽略列但是包含其他信息，内联速度快）

生产环境：需要考虑源代码要不要隐藏，调试要不要更友好

    内联会让代码体积变大，所以在生产环境不用内联
    隐藏源代码
    
    nosources-source-map 全部隐藏
    
    hidden-source-map 只隐藏源代码，会提示构建后代码错误信息
    
    最终得出最好的两种方案 --> source-map（最完整） / cheap-module-souce-map（错误提示一整行忽略列）
```

### oneOf

```js
匹配到 loader 后就不再向后进行匹配，优化生产环境的打包构建速度

如果有多个loader执行同一种文件 需要放在放在rules中

module:{
  rules:[
    {
      // js 语法检查
      test: /\.js$/,
      exclude: /node_modules/,
      // 优先执行
      enforce: 'pre',
      loader: 'eslint-loader',
      options: {
        fix: true
      }
    },
    {
      oneOf:[
        // 这里配置loader...
           {
          // js 兼容性处理
          test: /\.js$/,
          exclude: /node_modules/,
          loader: 'babel-loader',
          options: {
            presets: [
              [
                '@babel/preset-env',
                {
                  useBuiltIns: 'usage',
                  corejs: {version: 3},
                  targets: {
                    chrome: '60',
                    firefox: '50'
                  }
                }
              ]
            ]
          }
        },
      ]
    }
  ]
}
```

### 缓存

```js
babel 缓存:
	类似 HMR，将 babel 处理后的资源缓存起来（哪里的js改变就更新哪里，其他js还是用之前缓存的资源）,让第二次打包构建速度更快

文件资源缓存:
	1. hash: 每次 wepack 打包时会生成一个唯一的 hash 值
	   问题：重新打包，所有文件的 hsah 值都改变，会导致所有缓存失效。（可能只改动了一个文件）
	2. chunkhash：根据 chunk 生成的 hash 值。来源于同一个 chunk的 hash 值一样
	   问题：js 和 css 来自同一个chunk，hash 值是一样的（因为 css-loader 会将 css 文件加载到 js 中，所以同属于一个chunk）
	3. contenthash: 根据文件的内容生成 hash 值。不同文件 hash 值一定不一样(文件内容修改，文件名里的 hash 才会改变)
	   修改 css 文件内容，打包后的 css 文件名 hash 值就改变，而 js 文件没有改变 hash 值就不变，这样 css 和 js 缓存就会分开判断要不要重新请求资源 --> 让代码上线运行缓存更好使用
	   
output: {
  filename: 'js/build[contenthash:10].js',
  path: path.resolve(__dirname, 'build')
}, 
new MiniCssExtractPlugin({
  // 重名名提取出的css样式文件
  filename: 'css/main[contenthash:10].css'
}),
```

### tree shaking（树摇）

```js
前提：1. 必须使用 ES6 模块化 2. 开启 production 环境 （这样就自动会把无用代码去掉）
	mode:'production'
作用：减少代码体积

在 package.json 中配置：
"sideEffects": false 表示所有代码都没有副作用（都可以进行 tree shaking）
这样会导致的问题：可能会把通过 es6模块化引入但没有使用的文件 干掉（副作用）
所以可以配置："sideEffects": ["*.css", "*.less"] 不会对css/less文件tree shaking处理
```

### code split（代码分割）

````js
将打包输出的一个大的 bundle.js 文件拆分成多个小文件，这样可以并行加载多个文件，比加载一个文件更快
1和2可以结合使用	2和3可以结合使用

1. 多入口拆分:
entry: {
  // 多入口：有一个入口，最终输出就有一个bundle
  index: './src/js/index.js',
  test: './src/js/test.js'
},
output: {
  // [name]：取文件名
  filename: 'js/[name].[contenthash:10].js',
  path: resolve(__dirname, 'build')
},
    
2. optimization:
     optimization: {
       splitChunks: {
         chunks: 'all'
     }
   },
将 node_modules 中的代码单独打包（大小超过30kb）
自动分析多入口chunk中，有没有公共的文件。如果有会打包成单独一个chunk(比如两个模块中都引入了jquery会被打包成单独的文件)（大小超过30kb）

3. import 动态导入语法:
import(/* webpackChunkName:'kai' */ './kai')
  .then(res => {
    console.log(res.kai);
  })
  .catch(err => {
    console.log(err);
  });
/*
  通过js代码，让某个文件被单独打包成一个chunk
  import动态导入语法：能将某个文件单独打包(test文件不会和index打包在同一个文件而是单独打包)
  webpackChunkName:指定test单独打包后文件的名字
*/
````

### lazy loading（懒加载/预加载）

```js
1. 懒加载：当文件需要使用时才加载（需要代码分割）。但是如果资源较大，加载时间就会较长，有延迟
2. 正常加载：可以认为是并行加载（同一时间加载多个文件）没有先后顺序，先加载了不需要的资源就会浪费时间
3. 预加载 prefetch（兼容性很差）：会在使用之前，提前加载。等其他资源加载完毕，浏览器空闲了，再偷偷加载这个资源。这样在使用时已经加载好了，速度很快。所以在懒加载的基础上加上预加载会更好。

document.getElementById('btn').addEventListener('click', () => {
  // 将import的内容放在异步回调函数中使用，点击按钮，test.js才会被加载(不会重复加载)
  // webpackPrefetch: true表示开启预加载
  import(/* webpackChunkName:'test',webpackPrefetch:true */ './test')
    .then(({ sum }) => {
      console.log(sum(1, 2, 3, 4, 5));
    })
    .catch(err => {
      console.log(err);
    });
});
```

### pwa（离线可访问技术）

```js
离线可访问技术（渐进式网络开发应用程序）, 使用 serviceworker 和 workbox 技术, 优点是离线也能访问, 缺点是兼容性差

npm i workbox-webpack-plugin -D

const WorkboxWebpackPlugin = require('workbox-webpack-plugin'); // 引入插件

// 1. 注册插件
plugins: [
  new WorkboxWebpackPlugin.GenerateSW({
    /*
      1. 帮助serviceworker快速启动
      2. 删除旧的 serviceworker
      生成一个 serviceworker 配置文件
    */
    clientsClaim: true,
    skipWaiting: true
  })
]

// 2. index.js 入口文件中还需要写一段代码来激活它的使用
    /*
      1. eslint不认识 window、navigator全局变量
         解决：需要修改package.json中eslintConfig配置
         "env": {
           "browser": true // 支持浏览器端全局变量
         }
      2. sw代码必须运行在服务器上
    */
if ('serviceWorker' in navigator) {
  window.addEventListener('load', () => {
    navigator.serviceWorker
      .register('/service-worker.js')
      .then(() => {
        console.log('serviceWorker注册成功');
      })
      .catch(err => {
        console.log(err);
      });
  });
}
```

### 多进程打包

```js
多进程打包：某个任务消耗时间较长会卡顿，多进程可以同一时间干多件事，效率更高。
优点是提升打包速度，缺点是每个进程的开启和交流都会有开销（babel-loader消耗时间最久，所以使用thread-loader针对其进行优化）

npm i thread-loader -D

{
  test: /\.js$/,
  exclude: /node_modules/,
  use: [
    /* 
      thread-loader会对其后面的loader（这里是babel-loader）开启多进程打包。 
      进程启动大概为600ms，进程通信也有开销。(启动的开销比较昂贵，不要滥用)
      只有工作消耗时间比较长，才需要多进程打包
    */
    {
      loader: 'thread-loader',
      options: {
        workers: 2 // 进程2个
      }
    },
    {
      loader: 'babel-loader',
      options: {
        presets: [
          [
            '@babel/preset-env',
            {
              useBuiltIns: 'usage',
              corejs: { version: 3 },
              targets: {
                chrome: '60',
                firefox: '50'
              }
            }
          ]
        ],
        // 开启babel缓存
        // 第二次构建时，会读取之前的缓存
        cacheDirectory: true
      }
    }
  ]
},
```

### externals

````js
externals：让某些库不打包，通过 cdn 引入

webpack.config.js 中配置:
externals: {
  // 拒绝jQuery被打包进来(通过cdn引入，速度会快一些)
  // 忽略的库名 -- npm包名
  jquery: 'jQuery'
}
需要在 index.html 中通过 cdn 引入：
<script src="https://cdn.bootcss.com/jquery/1.12.4/jquery.min.js"></script>
````

### dll

```js
让某些库单独打包，后直接引入到 build 中。可以在 code split 分割出 node_modules 后再用 dll 更细的分割，优化代码运行的性能

webpack.dll.js 配置：(将 jquery 单独打包)
/*
  node_modules的库会打包到一起，但是很多库的时候打包输出的js文件就太大了
  使用dll技术，对某些库（第三方库：jquery、react、vue...）进行单独打包
  当运行webpack时，默认查找webpack.config.js配置文件
  需求：需要运行webpack.dll.js文件
    --> webpack --config webpack.dll.js（运行这个指令表示以这个配置文件打包）
*/
const { resolve } = require('path');
const webpack = require('webpack');

module.exports = {
  entry: {
    // 最终打包生成的[name] --> jquery
    // ['jquery] --> 要打包的库是jquery
    jquery: ['jquery']
  },
  output: {
    // 输出出口指定
    filename: '[name].js', // name就是jquery
    path: resolve(__dirname, 'dll'), // 打包到dll目录下
    library: '[name]_[hash]', // 打包的库里面向外暴露出去的内容叫什么名字
  },
  plugins: [
    // 打包生成一个manifest.json --> 提供jquery的映射关系（告诉webpack：jquery之后不需要再打包和暴露内容的名称）
    new webpack.DllPlugin({
      name: '[name]_[hash]', // 映射库的暴露的内容名称
      path: resolve(__dirname, 'dll/manifest.json') // 输出文件路径
    })
  ],
  mode: 'production'
};

webpack.config.js 配置：(告诉 webpack 不需要再打包 jquery，并将之前打包好的 jquery 跟其他打包好的资源一同输出到 build 目录下)
npm i add-asset-html-webpack-plugin -D

// 引入插件
const webpack = require('webpack');
const AddAssetHtmlWebpackPlugin = require('add-asset-html-webpack-plugin');

// plugins中配置：
plugins: [
  new HtmlWebpackPlugin({
    template: './src/index.html'
  }),
  // 告诉webpack哪些库不参与打包，同时使用时的名称也得变
  new webpack.DllReferencePlugin({
    manifest: resolve(__dirname, 'dll/manifest.json')
  }),
  // 将某个文件打包输出到build目录下，并在html中自动引入该资源
  new AddAssetHtmlWebpackPlugin({
    filepath: resolve(__dirname, 'dll/jquery.js')
  })
]
```

### Webpack 配置详情

#### 1.  entry

```js
1. string --> './src/index.js'，单入口
   打包形成一个 chunk  输出一个 bundle 文件 此时 chunk 的名称默认是 main
   
2. array --> ['./src/index.js', './src/add.js']，多入口
   所有入口文件最终只会形成一个 chunk 输出出去只有一个 bundle 文件（一般只用在 HMR 功能中让 html 热更新生效）
   
3. object，多入口
   有几个入口文件就形成几个 chunk，输出几个 bundle 文件，此时 chunk 的名称是 key 值
   
--> 特殊用法:
entry: {
  // 最终只会形成一个chunk, 输出出去只有一个bundle文件。
  index: ['./src/index.js', './src/count.js'], 
  // 形成一个chunk，输出一个bundle文件。
  add: './src/add.js'
}
```

#### 2. output

```js
output: {
  // 文件名称（指定名称+目录）
  filename: 'js/[name].js',
  // 输出文件目录（将来所有资源输出的公共目录）
  path: resolve(__dirname, 'build'),
  // 所有资源引入公共路径前缀 --> 'imgs/a.jpg' --> '/imgs/a.jpg'
  publicPath: '/',
  chunkFilename: 'js/[name]_chunk.js', // 指定非入口chunk的名称
  library: '[name]', // 打包整个库后向外暴露的变量名
  libraryTarget: 'window' // 变量名添加到哪个上 browser：window
  // libraryTarget: 'global' // node：global
  // libraryTarget: 'commonjs' // conmmonjs模块 exports
},
```

#### 3. module

```js
module: {
  rules: [
    // loader的配置
    {
      test: /\.css$/,
      // 多个loader用use
      use: ['style-loader', 'css-loader']
    },
    {
      test: /\.js$/,
      // 排除node_modules下的js文件
      exclude: /node_modules/,
      // 只检查src下的js文件
      include: resolve(__dirname, 'src'),
      enforce: 'pre', // 优先执行
      // enforce: 'post', // 延后执行
      // 单个loader用loader
      loader: 'eslint-loader',
      options: {} // 指定配置选项
    },
    {
      // 以下配置只会生效一个
      oneOf: []
    }
  ]
},
```

#### 4. resolve

````js
// 解析模块的规则
resolve: {
  // 配置解析模块路径别名: 优点：当目录层级很复杂时，简写路径；缺点：路径不会提示
  alias: {
    $css: resolve(__dirname, 'src/css')
  },
  // 配置省略文件路径的后缀名（引入时就可以不写文件后缀名了）
  extensions: ['.js', '.json', '.jsx', '.css'],
  // 告诉 webpack 解析模块应该去找哪个目录
  modules: [resolve(__dirname, '../../node_modules'), 'node_modules']
}
````

#### 5. dev server

````js
1. 其中，跨域问题：同源策略中不同的协议、端口号、域名就会产生跨域。
2. 正常的浏览器和服务器之间有跨域，但是服务器之间没有跨域。代码通过代理服务器运行，所以浏览器和代理服务器之间没有跨域，浏览器把请求发送到代理服务器上，代理服务器替你转发到另外一个服务器上，服务器之间没有跨域，所以请求成功。代理服务器再把接收到的响应响应给浏览器。这样就解决开发环境下的跨域问题

devServer: {
  // 运行代码所在的目录
  contentBase: resolve(__dirname, 'build'),
  // 监视contentBase目录下的所有文件，一旦文件变化就会reload
  watchContentBase: true,
  watchOptions: {
    // 忽略文件
    ignored: /node_modules/
  },
  // 启动gzip压缩
  compress: true,
  // 端口号
  port: 5000,
  // 域名
  host: 'localhost',
  // 自动打开浏览器
  open: true,
  // 开启HMR功能
  hot: true,
  // 不要显示启动服务器日志信息
  clientLogLevel: 'none',
  // 除了一些基本信息外，其他内容都不要显示
  quiet: true,
  // 如果出错了，不要全屏提示
  overlay: false,
  // 服务器代理，--> 解决开发环境跨域问题
  proxy: {
    // 一旦devServer(5000)服务器接收到/api/xxx的请求，就会把请求转发到另外一个服务器3000
    '/api': {
      target: 'http://localhost:3000',
      // 发送请求时，请求路径重写：将/api/xxx --> /xxx （去掉/api）
      pathRewrite: {
        '^/api': ''
      }
    }
  }
}
````

#### 6. optimization

```js
1. contenthash 缓存会导致一个问题：修改 a 文件导致 b 文件 contenthash 变化。
2. 因为在 index.js 中引入 a.js，打包后 index.js 中记录了 a.js 的 hash 值，而 a.js 改变，其重新打包后的 hash 改变，导致 index.js 文件内容中记录的 a.js 的 hash 也改变，从而重新打包后 index.js 的 hash 值也会变，这样就会使缓存失效。（改变的是a.js文件但是 index.js 文件的 hash 值也改变了）
解决办法：runtimeChunk --> 将当前模块记录其他模块的 hash 单独打包为一个文件 runtime，这样 a.js 的 hash 改变只会影响 runtime 文件，不会影响到 index.js 文件

output: {
  filename: 'js/[name].[contenthash:10].js',
  path: resolve(__dirname, 'build'),
  chunkFilename: 'js/[name].[contenthash:10]_chunk.js' // 指定非入口文件的其他chunk的名字加_chunk
},
optimization: {
  splitChunks: {
    chunks: 'all',
    /* 以下都是splitChunks默认配置，可以不写
    miniSize: 30 * 1024, // 分割的chunk最小为30kb（大于30kb的才分割）
    maxSize: 0, // 最大没有限制
    minChunks: 1, // 要提取的chunk最少被引用1次
    maxAsyncRequests: 5, // 按需加载时并行加载的文件的最大数量为5
    maxInitialRequests: 3, // 入口js文件最大并行请求数量
    automaticNameDelimiter: '~', // 名称连接符
    name: true, // 可以使用命名规则
    cacheGroups: { // 分割chunk的组
      vendors: {
        // node_modules中的文件会被打包到vendors组的chunk中，--> vendors~xxx.js
        // 满足上面的公共规则，大小超过30kb、至少被引用一次
        test: /[\\/]node_modules[\\/]/,
        // 优先级
        priority: -10
      },
      default: {
        // 要提取的chunk最少被引用2次
        minChunks: 2,
        prority: -20,
        // 如果当前要打包的模块和之前已经被提取的模块是同一个，就会复用，而不是重新打包
        reuseExistingChunk: true
      }
    } */
  },
  // 将index.js记录的a.js的hash值单独打包到runtime文件中
  runtimeChunk: {
    name: entrypoint => `runtime-${entrypoint.name}`
  },
  minimizer: [
    // 配置生产环境的压缩方案：js/css
    new TerserWebpackPlugin({
      // 开启缓存
      cache: true,
      // 开启多进程打包
      parallel: true,
      // 启用sourceMap(否则会被压缩掉)
      sourceMap: true
    })
  ]
}
```



