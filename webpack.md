#配置说明：

命令行 npm run dev

"scripts": {
  "dev": "node build/dev-server.js",
  "build": "node build/build.js",
  "unit": "karma start test/unit/karma.conf.js --single-run",
  "e2e": "node test/e2e/runner.js",
  "test": "npm run unit && npm run e2e",
  "lint": "eslint --ext .js,.vue src test/unit/specs test/e2e/specs"
},
npm run XXX, 敲入不同的命令，执行不同的js文件
根据package.json文件，运行scripts属性的dev，即运行build/dev-server.js;

#webpack相关的配置文件：
webpack.base.conf.js：基础配置，此模块会被以下两个模块依赖
webpack.dev.conf.js：开发阶段的webpack配置，比如热替换，css注入style标签等
webpack.prod.conf.js：打包生产环境模块的配置, 比如压缩js，提取css到文件等

参考 https://x-front-team.github.io/2016/08/14/webpack%E5%85%A5%E9%97%A8%E4%B8%8E%E5%AE%9E%E6%88%98/

##webpack.base.conf.js

1.entry
bundle的入口点。
如果传入一个字符串，这个字符串就会被解析为启动时加载的模块。
如果传入个数组，所有模块都是启动时加载，模块导出到最后一个里面。
entry: ["./entry1", "./entry2"]
如果传入一个对象，就会创建多个输入包文件，对象键值就chunk的name，值可以是字符串或者是数组。

```
entry: {
    app: './src/main.js'
  },
或
entry: {
    page1: "./page1",
    page2: ["./entry1", "./entry2"]
},

```

2.output
output参数是个对象，用于定义构建后的文件的输出。其中包含path、filename和publicPath

```
output: {
    <!-- path：打包后的js文件存放地址。绝对路径 (required) -->
    path: config.build.assetsRoot,
    <!-- publicPath：在你网站运行时候可能你访问的资源地址（image，url等）可能不是你打包的目录（path）, publicPath 就是在 path 前加上一段；可以是目录，也可以是CDN地址。 -->
    <!-- publicPath指定了一个在浏览器中被引用的URL地址。 对于使用<script> 和 <link>加载器，当文件路径不同于他们的本地磁盘路径（由path指定）时候publicPath被用来作为href或者url指向该文件。这种做法在你需要将静态文件放在不同的域名或者CDN上面的时候是很有用的。 Webpack Dev Server 也是用这个方式来读取文件的。与path搭配使用上[hash]就可以做好缓存方案了。-->
    publicPath: process.env.NODE_ENV === 'production' ? config.build.assetsPublicPath : config.dev.assetsPublicPath,
    <!-- 指定输出到硬盘的文件的的文件名。这里不能是一个绝对的路径！output.path会确定该文件的存在硬盘额路径的。filename仅仅用来给每个文件命名而已。 -->
    filename: '[name].js'
  },

  或
  output: {
      path: "/home/proj/public/assets",
      publicPath: "/assets/"
  }
  index.html引用
  <head>
    <link href="/assets/spinner.gif"/>
  </head>

```

使用CDN 和 hash的例子.（有待研究）
参考 https://zhuanlan.zhihu.com/p/21346555


3.module

关于模块的加载相关，我们就定义在module.loaders中。这里通过正则表达式去匹配不同后缀的文件名，然后给它们定义不同的加载器。比如说给less文件定义串联的三个加载器（！用来定义级联关系）：
1、loader
loader是webpack中比较重要的部分，她是处理各类资源的执行者。它们是一系列的函数（运行在node.js中），将资源中的代码作为参数，然后返回新的代码。你可以用loader告诉webpack可以加载哪些文件，或者不加载哪些文件。

module.loaders
自动引用的加载器的数组.
每个元素有这些选项:
a: test: 必须满足的条件
b: exclude: 不满足的条件
c: include: 必须满足条件
d: loader: 用 "!" 隔开多个loader
e: loaders: 多个loader

Loader的特点：
    可以链式执行。它们在一个管道中被提交，只需要保证最后的loader返回JavaScript即可，其他loader可以返回任意方便下一个loader处理的内容。
    可以异步or同步执行
    运行在Node.js中，可以做几乎任何事儿
    可以接收query参数，用于向loader传递参数
    配置中可与正则/扩展结合使用
    可以在npm中发布并使用
    除了main,其他模块可以导出成loader
    可以通过配置调入
    和插件（plugins）配合可获得更多功能
    可生成其他格式文件

```
modules: {
    loaders: [
        {
            test: /\.js$/, //匹配希望处理文件的路径
            include: projectRoot,// 匹配希望处理文件的路径
            exclude: /node_modules/, // 匹配不希望处理文件的路径
            loaders: 'xxx-loader?a=x&b=y'  //此处xxx-loader 可以简写成xxx , ？后以query方式传递给loader参数
        },
        ...
    ]
}

module: {
    loaders: [
      {
        test: /\.vue$/,
        loader: 'vue'
      },
      {
        test: /\.js$/,
        loader: 'babel',
        include: projectRoot,
        exclude: /node_modules/
      },
      {
        test: /\.json$/,
        loader: 'json'
      },
      {
      <!-- 添加用来定义png、jpg这样的图片资源在小于10k时自动处理为base64图片的加载器 -->
        test: /\.(png|jpe?g|gif|svg)(\?.*)?$/,
        loader: 'url',
        query: {
          limit: 10000,
          name: utils.assetsPath('img/[name].[hash:7].[ext]')
        }
      },
      {
        test: /\.(woff2?|eot|ttf|otf)(\?.*)?$/,
        loader: 'url',
        query: {
          limit: 10000,
          name: utils.assetsPath('fonts/[name].[hash:7].[ext]')
        }
      }
},

```
4、resolve
webpack在构建包的时候会按目录的进行文件的查找，resolve属性中的extensions数组中用于配置程序可以自行补全哪些文件后缀,应用于解决模块的扩展数组, 配置后, 数组里的扩展名在require时可以省略：

```
resolve: {
    extensions: ['', '.js', '.vue', '.json'],
    <!-- webpack没有在resolve.root 或者resolve.modulesDirectories找到的模块的一个目录（或者目录绝对路径的数组）. -->
    fallback: [path.join(__dirname, '../node_modules')],
    <!-- 设置别名 -->
    <!-- 设置别名后, 你想require('src/scripts/app.js')只需要写成require('js/app.js')即可 -->
    alias: {
      'vue$': 'vue/dist/vue.common.js',
      'src': path.resolve(__dirname, '../src'),
      'assets': path.resolve(__dirname, '../src/assets'),
      'components': path.resolve(__dirname, '../src/components')
    }
  },

```

然后我们想要加载一个js文件时，只要require(‘common’)就可以加载common.js文件了。

5、resolveLoader
该配置和resolve类似, 只不过是作用于loaders。
参考 http://stephenzhao.github.io/2016/06/13/webpack-doc-configuration/

6、eslint

```
eslint: {
  // 以更友好的格式输出eslint错误信息
  formatter: require('eslint-friendly-formatter')
},
```

7、vue

```
// vue-loader配置
vue: {
  loaders: utils.cssLoaders({ sourceMap: useCssSourceMap }),
  postcss: [
    require('autoprefixer')({
      browsers: ['last 2 versions']
    })
  ]
```

##webpack.dev.conf.js

```
var config = require('../config')
var webpack = require('webpack')
var merge = require('webpack-merge')
var utils = require('./utils')
// 引入基础配置
var baseWebpackConfig = require('./webpack.base.conf')
// 一个webpack插件,用于将输出的模块,自动生成script,link等标签插入html,以及设置标题等
var HtmlWebpackPlugin = require('html-webpack-plugin')

// add hot-reload related code to entry chunks
Object.keys(baseWebpackConfig.entry).forEach(function (name) {
  baseWebpackConfig.entry[name] = ['./build/dev-client'].concat(baseWebpackConfig.entry[name])
})

module.exports = merge(baseWebpackConfig, {
  module: {
    loaders: utils.styleLoaders({ sourceMap: config.dev.cssSourceMap })
  },
  // eval-source-map is faster for development
  //选择开发工具来提高debug效率.
  devtool: '#eval-source-map',

  // 在基础配置的基础上继续添加插件
  plugins: [
    // 设置前端级别的环境变量,从node借鉴过来,非常好用的特性,可以用来区分开发模式或生产模式
    // 比如可以在开发模式下,快速生成表单数据,方便测试
    new webpack.DefinePlugin({
      // 从node环境变量读取NODE_ENV传到webpack模块中,就不需要改env.js来切换了,不然有时候会忘记改回去就提交了
      'process.env': config.dev.env
    }),
    // https://github.com/glenjamin/webpack-hot-middleware#installation--usage
    // 跟hot-reload有关,按顺序加载模块
    new webpack.optimize.OccurrenceOrderPlugin(),
    // hot-reload模块
    new webpack.HotModuleReplacementPlugin(),
    new webpack.NoErrorsPlugin(),
    // https://github.com/ampedandwired/html-webpack-plugin
    new HtmlWebpackPlugin({
      // 输出入口页面文件名
      filename: 'index.html',
      // 指定入口页面模板位置
      template: 'index.html',
      // 是否插入script等标签
      inject: true
    })
  ]
})

```


#webpack.prod.conf.js

```
var path = require('path')
var config = require('../config')
var utils = require('./utils')
var webpack = require('webpack')
var merge = require('webpack-merge')
// 引入基础配置
var baseWebpackConfig = require('./webpack.base.conf')
// 用于提取css文件的webpack插件
var ExtractTextPlugin = require('extract-text-webpack-plugin')
// 一个webpack插件,用于将输出的模块,自动生成script,link等标签插入html,以及设置标题等
var HtmlWebpackPlugin = require('html-webpack-plugin')
var env = process.env.NODE_ENV === 'testing'
  ? require('../config/test.env')
  : config.build.env

var webpackConfig = merge(baseWebpackConfig, {
  module: {
    loaders: utils.styleLoaders({ sourceMap: config.build.productionSourceMap, extract: true })
  },
  //选择开发工具来提高debug效率.
  devtool: config.build.productionSourceMap ? '#source-map' : false,
  output: {
    // 静态资源的web绝对路径前缀
    path: config.build.assetsRoot,
    // 输出的主文件名
    filename: utils.assetsPath('js/[name].[chunkhash].js'),
    // 输出的按需加载的文件名
    chunkFilename: utils.assetsPath('js/[id].[chunkhash].js')
  },
  vue: {
    loaders: utils.cssLoaders({
      sourceMap: config.build.productionSourceMap,
      extract: true
    })
  },
  plugins: [
    // http://vuejs.github.io/vue-loader/en/workflow/production.html
    new webpack.DefinePlugin({
      'process.env': env
    }),
    // 压缩js
    new webpack.optimize.UglifyJsPlugin({
      compress: {
        warnings: false
      }
    }),
    new webpack.optimize.OccurrenceOrderPlugin(),
    // extract css into its own file
    // 提取CSS文件, [name]为相应的entry的key, [contenthash:8]为8位文件内容hash
    new ExtractTextPlugin(utils.assetsPath('css/[name].[contenthash].css')),
    // generate dist index.html with correct asset hash for caching.
    // you can customize output by editing /index.html
    // see https://github.com/ampedandwired/html-webpack-plugin
    new HtmlWebpackPlugin({
      filename: process.env.NODE_ENV === 'testing'
        ? 'index.html'
        : config.build.index,
      template: 'index.html',
      inject: true,
      minify: {
        removeComments: true,
        collapseWhitespace: true,
        removeAttributeQuotes: true
        // more options:
        // https://github.com/kangax/html-minifier#options-quick-reference
      },
      // necessary to consistently work with multiple chunks via CommonsChunkPlugin
      chunksSortMode: 'dependency'
    }),
    // split vendor js into its own file
    new webpack.optimize.CommonsChunkPlugin({
      name: 'vendor',
      minChunks: function (module, count) {
        // any required modules inside node_modules are extracted to vendor
        return (
          module.resource &&
          /\.js$/.test(module.resource) &&
          module.resource.indexOf(
            path.join(__dirname, '../node_modules')
          ) === 0
        )
      }
    }),
    // extract webpack runtime and module manifest to its own file in order to
    // prevent vendor hash from being updated whenever app bundle is updated
    new webpack.optimize.CommonsChunkPlugin({
      name: 'manifest',
      chunks: ['vendor']
    })
  ]
})

if (config.build.productionGzip) {
  var CompressionWebpackPlugin = require('compression-webpack-plugin')

  webpackConfig.plugins.push(
    new CompressionWebpackPlugin({
      asset: '[path].gz[query]',
      algorithm: 'gzip',
      test: new RegExp(
        '\\.(' +
        config.build.productionGzipExtensions.join('|') +
        ')$'
      ),
      threshold: 10240,
      minRatio: 0.8
    })
  )
}

module.exports = webpackConfig

```
