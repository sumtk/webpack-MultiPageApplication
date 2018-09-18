# webpack构建性能优化

## CommonsChunkPlugin

```text
new webpack.optimize.CommonsChunkPlugin({
      name: 'vendor',
      chunks: chunks,
      minChunks: 4 || chunks.length 
    }),
    new webpack.optimize.CommonsChunkPlugin({
      name: 'manifest',
      chunks: ['vendor']
    })
```

## DllPlugin

#### webpack.dll.config.js

```text
const path = require('path');
const webpack = require('webpack');
const UglifyJsPlugin = require('uglifyjs-webpack-plugin');
const config = require('../config');

module.exports = {
    entry: {
        vendor: ['vue/dist/vue.esm.js', 'vuex', 'axios', 'vue-router', 'babel-polyfill', 'lodash'] // 所需要的打包前端公共模块
    },
    output: {
        path: path.join(__dirname, '../static/js'), // 打包后文件输出的位置
        filename: '[name].dll.js',
        /**
         * output.library
         * 将会定义为 window.${output.library}
         * 在这次的例子中，将会定义为`window.vendor_library`
         */
        library: '[name]_library'
    },
    plugins: [
        new webpack.DllPlugin({  //主要是使用这个插件去打包js
            /**
             * path
             * 定义 manifest 文件生成的位置
             * [name]的部分由entry的名字替换
             */
            path: path.join(__dirname, '.', '[name]-manifest.json'),
            /**
             * name
             * dll bundle 输出到那个全局变量上
             * 和 output.library 一样即可。
             */
            name: '[name]_library',
            context: path.join(__dirname, '..')
        }),
        new UglifyJsPlugin({    // 使用这个插件可以混淆打包完成的js
            uglifyOptions: {
                compress: {
                    warnings: false
                }
            },
            sourceMap: config.build.productionSourceMap,
            parallel: true
        })
    ]
};
```

然后在 package.json 中配置命令

```text
"scripts": {
    ...
    "build:dll": "webpack --config build/webpack.dll.conf.js"
}1234
```

执行 npm run build:dll 命令来生成 vendor.dll.js 和 vendor-manifest.json

或者执行 `webpack --config build/webpack.dll.config.js`，其中vendor.dll.js即合并打包后第三方模块。另外一个vendor-mainifest.json存储各个模块和所需公用模块的对应关系。

将第三方模块打完包以后，我们就需要使用DLLReferencePlugin来将它和我们的业务代码进行融合，我们修改`webpack.base.config(vue-cli生成配置)`，添加plugin如下：

```text
plugins: [
        new webpack.DllReferencePlugin({
            context: __dirname,  // 与DllPlugin中的那个context保持一致
            manifest: require('./vendor-manifest.json')
        }),
        ......
]
```

同时，我们还需要手动的将`vendor.dll.js`插入类似`index.html`这样的模板文件才可以生效

```text
<script src="/vendor.dll.js"></script>
```

## AutoDllPlugin

AutoDllPlugin这个插件相当于自动同时完成了`DllReferencePlugin`和`DllPlugin`的工作，只需要在`webpack.base.config`中添加

```text
plugins: [
    new AutoDllPlugin({
            inject: true, // will inject the DLL bundles to html
            context: path.join(__dirname, '..'),
            filename: '[name]_[hash].dll.js',
            path: 'res/js',
            plugins: mode === 'online' ? [
                new UglifyJsPlugin({
                    uglifyOptions: {
                        compress: {
                            warnings: false
                        }
                    },
                    sourceMap: config.build.productionSourceMap,
                    parallel: true
                })
            ] : [],
            entry: {
                vendor: ['vue/dist/vue.esm.js', 'vuex', 'axios', 'vue-router', 'babel-polyfill', 'lodash']
            }
     })
]
复制代码
```

，不需要额外的`webpack.dll.config.js`配置以及不需要手动将打完好的包拷贝到对应的模板文件中。

## 多线程构建

webpack和其他大部分js工具相同都是单线程对项目进行处理，  
然而 Webpack 这个工具强就强在流程设计的扩展性如此之强，可以人为的加上多进程处理。  
其在编译文件流程如下：

```text
1. 开始编译 (Compiler#run)
2. 开始编译入口文件 (Compilation#addEntry)
    2.1 开始编译文件 (Compilation#buildModule => NormalModule#build)
    2.2 执行 Loader 得到文件结果 (NormalModule#runLoaders)
    2.3 根据结果解析依赖 (NormalModule#parser.parse)
    2.4 处理依赖文件列表 (Compilation#processModuleDependencies)
    2.5 开始编译每个依赖文件 (异步，从这里开始递归操作: 编译文件->解析依赖->编译依赖文件->解析深层依赖...)
复制代码
```

这里的关键在于递归操作 2.5 开始编译每个依赖文件 这一步是异步设计，每个依赖文件的编译彼此之间互不影响。不过虽然是异步的，但还是跑在一个线程里。但是这样的设计却带来了多进程的可行性。

编译文件中主要的耗时操作在于 Loader 对源文件的转换操作，而 Loader 的可异步的设计使得转换操作的执行并不被限制在同一线程内。下面对 Loader 进行改造，使其支持多进程并发：

```text
2.2 执行 Loader 得到文件结果
    LoaderWrapper 作为新的 Loader 入口接收文件输入信息
    LoaderWrapper 创建一个子进程 (child_process#fork) (这一步可维护一个进程池)
    子进程中，通过调用原始 Loader，转换输入文件，然后把最终结果传递给父进程
    父进程将收到的结果作为 Loader 结果传递给 Webpack
复制代码
```

HappyPack 的实现就是这个流程，我们来使用babel-loader作为例子，来讲解一下HappyPack如何配置

通常情况下，我们使用的`babel-loader`如下所示

```text
webpack.base.config.js
...
module: {
    rules: [
      ...
      {
        test: /\.js$/,
        include: [resolve('src'), resolve('lib'),resolve('test'), resolve('node_modules/webpack-dev-server/client')], // 通过合理配置include也可以对提升构建性能
        use: [
          {
            loader: 'babel-loader'
          },
        ],
        exclude: /node_modules/ // 通过合理配置exclude也可以对提升构建性能
      }
}
复制代码
```

转换成HappyPack，配置改写为

```text
const HappyPack = require('happypack');
const happyThreadPool = HappyPack.ThreadPool({size: os.cpus().length});

// 省略其他配置
module.exports = {
    module: {
        rules: [
            {
                test: /\.js$/,
                include: [resolve('src'), resolve('lib'), resolve('test'), resolve('node_modules/webpack-dev-server/client')],
                use: [
                    {
                        loader: 'happypack/loader?id=happybabel'  // 将loader换成happypack并将id指向插件id参数
                    },
                ],
                exclude: /node_modules/
            }
        ]
    },
   plugins: [
        new HappyPack({  // HappyPack插件
            id: 'happybabel',
            loaders: ['babel-loader?cacheDirectory=true'],
            threadPool: happyThreadPool,
        })
    ]
}
复制代码
```

HappyPack不只可以对babel-loader进行处理，其他vue-loader,css-loader等都可以用他进行加速优化，只需要如上增加实例以及改写loader即可。使用HappyPack整体优化后，在我们的项目中，构建速度基本可以提高70%。

## html-webpack-plugin-for-multihtml

```text
npm install html-webpack-plugin-for-multihtml -save-dev
```

```text
const HtmlWebpackPlugin = require('html-webpack-plugin-for-multihtml');
// 省略其他代码

plugins:[
  new HtmlWebpackPlugin({
          template: filePath,
          filename: `${filename}.html`,
          chunks: ['manifest', 'vendor', filename],
          inject: true,
          multihtmlCache: true  // 增加该配置
  })
]
```

