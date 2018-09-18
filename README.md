# 关于webpack构建多页面应用与转化为多包独立应用的相关配置及webpack构建性能优化

## 前言

#### 学习使用webpack的目的是什么？

* 使用 ES6 进行开发
* 模块化开发
* 自动压缩合并 CSS 和 JS 文件
* 使用 ESLint 进行代码检查
* 自动生成 HTML 文件
* 自动抽取 CSS 文件
* 样式去重
* ···

#### 通过这个分享我能掌握什么？

* 单页、多页项目的基本构造
* webpack的基本配置
* vue组件的基本构造与应用
* 多页与单页的差别
* 项目热重载的基本性能优化
* 根据自身需求定制化配置的思考
* ···

## 为什么需要多页面应用

* 项目比较大，无法进行全局的把握
* 项目需要多次的更新迭代\(**Important\)**
* SEO的需要
* ...

## 配置

**webpack.config.js**

```javascript
const path = require('path');

module.exports = {
  entry: "./app/entry", // string | object | array  
  entry: ["./app/entry1", "./app/entry2"],
  entry: {
    a: "./app/entry-a",
    b: ["./app/entry-b1", "./app/entry-b2"]
  },
  // 这里应用程序开始执行
  // webpack 开始打包

  output: {
    // webpack 如何输出结果的相关选项

    path: path.resolve(__dirname, "dist"), // string
    // 所有输出文件的目标路径
    // 必须是绝对路径（使用 Node.js 的 path 模块）

    filename: "bundle.js", // string    
    filename: "[name].js", // 用于多个入口点(entry point)（出口点？）
    filename: "[chunkhash].js", // 用于长效缓存
    // 「入口分块(entry chunk)」的文件名模板（出口分块？）

    publicPath: "/assets/", // string    
    publicPath: "",
    publicPath: "https://cdn.example.com/",
    // 输出解析文件的目录，url 相对于 HTML 页面

  module: {
    // 关于模块配置

    rules: [
      // 模块规则（配置 loader、解析器等选项）

      {
        test: /\.jsx?$/,
        include: [
          path.resolve(__dirname, "app")
        ],
        exclude: [
          path.resolve(__dirname, "app/demo-files")
        ],
        // 这里是匹配条件，每个选项都接收一个正则表达式或字符串
        // test 和 include 具有相同的作用，都是必须匹配选项
        // exclude 是必不匹配选项（优先于 test 和 include）
        // 最佳实践：
        // - 只在 test 和 文件名匹配 中使用正则表达式
        // - 在 include 和 exclude 中使用绝对路径数组
        // - 尽量避免 exclude，更倾向于使用 include

        loader: "babel-loader",
        // 应该应用的 loader，它相对上下文解析
        // 为了更清晰，`-loader` 后缀在 webpack 2 中不再是可选的
        // 查看 webpack 1 升级指南。

        options: {
          presets: ["es2015"]
        },
        // loader 的可选项
      },

      {
        test: /\.html$/,
        test: "\.html$"

        use: [
          // 应用多个 loader 和选项
          "htmllint-loader",
          {
            loader: "html-loader",
            options: {
              /* ... */
            }
          }
        ]
      }
    ],

  resolve: {
    // 解析模块请求的选项
    // （不适用于对 loader 解析）

    modules: [
      "node_modules",
      path.resolve(__dirname, "app")
    ],
    // 用于查找模块的目录

    extensions: [".js", ".json", ".jsx", ".css"],
    // 使用的扩展名

    alias: {
      // 模块别名列表

      "module": "new-module",
      // 起别名："module" -> "new-module" 和 "module/path/file" -> "new-module/path/file"

      "only-module$": "new-module",
      // 起别名 "only-module" -> "new-module"，但不匹配 "only-module/path/file" -> "new-module/path/file"

      "module": path.resolve(__dirname, "app/third/module.js"),
      // 起别名 "module" -> "./app/third/module.js" 和 "module/file" 会导致错误
      // 模块别名相对于当前上下文导入
    },

  devtool: "source-map", 
  // enum  
  // 通过在浏览器调试工具(browser devtools)中添加元信息(meta info)增强调试
  // 牺牲了构建速度的 `source-map' 是最详细的。

  devServer: {
    proxy: { 
    // proxy URLs to backend development server
      '/api': 'http://localhost:3000'
    },
    compress: true, 
    // enable gzip compression
    hot: true 
    // hot module replacement. Depends on HotModuleReplacementPlugin
  },

  plugins: [
    // ...
  ],
  // 附加插件列表
```



