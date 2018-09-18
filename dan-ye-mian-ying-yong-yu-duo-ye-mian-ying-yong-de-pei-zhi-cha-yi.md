# 单页面应用与多页面应用的配置差异

## 入口文件配置：entry参数

```javascript
entry: "./app/entry", 
entry: ["./app/entry1", "./app/entry2"],
entry: {
    a: "./app/entry-a",
    b: ["./app/entry-b1", "./app/entry-b2"]
},
```

```text
var pageArr = [
    'index/login',
    'index/index',
    'alert/index',
  ];
  var configEntry = {};
  pageArr.forEach((page) => {
    configEntry[page] = path.resolve(pagesDir, page + '/page');
  });
```

## 输出文件：output参数

```text
output: {
    path: buildDir, // var buildDir = path.resolve(__dirname, './build');
    publicPath: '../../../../build/',
    filename: '[name]/entry.js',    // [name]表示entry每一项中的key，用以批量指定生成后文件的名称
    chunkFilename: '[id].bundle.js'
}
```

#### path {#articleHeader3}

path参数表示生成文件的根目录，需要传入一个**绝对路径**。path参数和后面的filename参数共同组成入口文件的完整路径。

#### publicPath {#articleHeader4}

publicPath参数表示的是一个URL路径（指向生成文件的根目录），用于生成css/js/图片/字体文件等资源的路径，以确保网页能正确地加载到这些资源。  
publicPath参数跟path参数的区别是：path参数其实是针对本地文件系统的，而publicPath则针对的是浏览器；因此，publicPath既可以是一个相对路径，如示例中的`'../../../../build/'`，也可以是一个绝对路径如`http://www.xxxxx.com/`。一般来说，我还是更推荐相对路径的写法，这样的话整体迁移起来非常方便。那什么时候用绝对路径呢？其实也很简单，当你的html文件跟其它资源放在不同的域名下的时候，就应该用绝对路径了，这种情况非常多见于后端渲染模板的场景。

#### filename {#articleHeader5}

filename属性表示的是如何命名生成出来的入口文件，规则有以下三种：

* \[name\]，指代入口文件的name，也就是上面提到的entry参数的key，因此，我们可以在name里利用`/`，即可达到控制文件目录结构的效果。
* \[hash\]，指代本次编译的一个hash版本，值得注意的是，只要是在同一次编译过程中生成的文件，这个\[hash\]的值就是一样的；在缓存的层面来说，相当于一次全量的替换。
* \[chunkhash\]，指代的是当前chunk的一个hash版本，也就是说，在同一次编译中，每一个chunk的hash都是不一样的；而在两次编译中，如果某个chunk根本没有发生变化，那么该chunk的hash也就不会发生变化。这在缓存的层面上来说，就是把缓存的粒度精细到具体某个chunk，只要chunk不变，该chunk的浏览器缓存就可以继续使用。

下面来说说如何利用filename参数和path参数来设计入口文件的目录结构，如示例中的`path: buildDir, // var buildDir = path.resolve(__dirname, './build');`和`filename: '[name]/entry.js'`，那么对于key为'index/login'的入口文件，生成出来的路径就是`build/index/login/entry.js`了，怎么样，是不是很简单呢？

#### chunkFilename {#articleHeader6}

chunkFilename参数与filename参数类似，都是用来定义生成文件的命名方式的，只不过，chunkFilename参数指定的是除入口文件外的chunk（这些chunk通常是由于webpack对代码的优化所形成的，比如因应实际运行的情况来异步加载）的命名。

## 各种Loader配置：module参数

```text
{
  test: /\.(png|jpe?g|gif|svg)(\?.*)?$/,
  loader: 'url-loader',
  options: {
    limit: 10000,
    name: utils.assetsPath('img/[name].[ext]')
  }
},
{
  test: /\.(mp4|webm|ogg|mp3|wav|flac|aac)(\?.*)?$/,
  loader: 'url-loader',
  options: {
    limit: 10000,
    name: utils.assetsPath('media/[name].[ext]')
  }
},
{
  test: /\.(woff2?|eot|ttf|otf)(\?.*)?$/,
  loader: 'url-loader',
  options: {
    limit: 10000,
    name: utils.assetsPath('fonts/[name].[ext]')
  }
}
```

## **添加额外功能：plugins参数**

{% hint style="info" %}
**结合实例说明**CommonsChunkPlugin、HtmlWebpackPlugin、ZipFilesPlugin
{% endhint %}

