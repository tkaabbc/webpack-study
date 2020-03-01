# webpack学习整理
### 什么是webpack？
  + 定义: webpack是一个模块打包工具。
  + 打个比方，对esMoudle的处理.
  + 假设在index.js中存在如下代码，如果把上面代码放在浏览器中，浏览器是不能解析的
  ```
    import A from './A';
    import B from './B';
    import C from './C';
  ```
  + 而webpack能够识别esMoudle，所以通过webapck对代码做构建
  ```
    npx webpack index.js
  ```
  + 可以把如下代码打包成浏览器能够解析的代码。
------
### 什么是模块打包工具？
  + webpack支持的模块：esMoudle、common.js、CMD、AMD
  ```
    esMoudle: import -> export default
    common.js: require -> moudle.export
    AMD: 使用require.js.
      // 定义模块 myModule.js
      define(id?, dependencies?, factory);
      id：可选参数，用来定义模块的标识，如果没有提供该参数，脚本文件名（去掉拓展名）
      dependencies：是一个当前模块依赖的模块名称数组
      factory：工厂方法，模块初始化要执行的函数或对象。如果为函数，它应该只被执行一次。
      例子：
      define('myModule', ['dependency'], function(){
          return {};
      });
      // 加载模块
      require([dependencies], function(){});
      例子：
      require(['myModule'], function (my){});
    CMD: 使用SeaJS
      // 定义模块  myModule.js
      define(function(require, exports, module) {
        module.exports = {
      });
      // 加载模块
      seajs.use(['myModule.js'], function(my){});
  ```
    + AMD和CDM区别
      通俗来说：
      AMD在加载完成定义（define）好的模块就会立即执行，所有执行完成后，遇到require才会执行主逻辑。（提前加载）
      CMD在加载完成定义（define）好的模块，仅仅是下载不执行，在遇到require才会执行对应的模块。（按需加载）
      AMD用户体验好，因为没有延迟，CMD性能好，因为只有用户需要的时候才执行。
  + webpack 最开始为js模块打包工具，后来随着发展webpack已经支持css、png/jpg....
------
### webpack的正确安装方式
  + [node安装](http://nodejs.cn/download "node安装")
  ```
    npm init 以node规范的形式创建一个包文件，webpack管理需要符合node规范
    npm install webpack webpack-cli -g 全局安装, 如果网络被墙，通过手机分享wifi热点, webpack-cli的作用就是让webpack命令可以在命令行中运行
    不推荐全局安装，如果项目webpack3的项目和webpack4的项目，那么全局安装就会有冲突。
    删除全局安装：npm uninstall webpack webpack-cli -g
    npm install webpack webpack-cli --save-dev(-D)
    npx webpack -v 此命令可以在当前项目的nodemoudles下找webpack
    npm info webpack 查看所有webpack版本
    npm webpack@4.2.1 安装webpack指定的4.2.1的版本
  ```
------
### webpack的配置文件
  + 创建webpack.config.js, 配置一个最基本的配置.
  ```
    const path = require('path');
    moudle.export = {
      mode: 'development(production)' development不压缩、production压缩、默认是production
      entry: {
        main: '入口文件'
      },
      output: {
        filename: '打包的文件名称',
        path: 输出路径-> path.resolve(__dirname, 'dist')
      }
    }
  ```
------
### 什么是loader?
  + 其实就是一个打包方案。它知道对于某些特定的文件知道怎么打包。而本身webpack并不知道对于这些文件应该怎么处理。
  + 打包图片配置。
  ```
    const jpgFile = require('img.jpg')
    moudle.export = {
      entry,
      output,
      module: {
        rules: [{
          test: '/\.jpg$/',
          use: {
            loader: 'file-loader'，
            options: {
              name: '[name][hash].[ext]'
            }
          }
        }]
      }
    }
    webpack在打包的时候如果遇到js的时候，webpack可以识别js打包。如果遇到jpg文件时候，把jpg打包交给file-loader去处理。
    file-loader做了什么呢？ 1.当filer-loader发现文件里引入了jpg文件，它会把文件移动到dist目录下，改一个名字，之后返回
    图片相对dist目录的地址。 然后把返回的地址，赋值给jpgFile变量。
  ```
------
### 使用loader打包静态资源-图片
  + 常用的loader打包配置
  ```
    moudle.export = {
      entry,
      output,
      module: {
        rules: [{
          test: '/\.(jpg|png|gif)$/',
          use: {
            loader: 'file-loader'，
            options: {
              name: '[name][hash].[ext]',
              outputPath: 'images/'
            }
          }
        }]
      }
    }
    [hash]代表输出添加hash、[ext]输出文件尾缀。
    outputPath可以在打包目录下生成images、里面放入打包的图片资源。
  ```
  + file-loader 可以 替换成 url-loader, url-loader作用是？
  相当于把图片生成base64字符串放在了打包的js中。 使用场景，是打包1-2kb的适合url-loader，可以减少一次http请求。
  ```
    rules: [{
      test: '/\.(jpg|png|gif)$/',
      use: {
        loader: 'url-loader'，
        options: {
          name: '[name][hash].[ext]',
          outputPath: 'images/',
          limit: 2048
        }
      }
    }]
    limit限制小于1kb的就打成js。
  ```
------
### 使用loader打包静态资源-样式
  + 一般打包css使用两个loader、 style-loader和css-loader，基本配置如下:
  ```
    import css from './index.css'
    rules: [{
      test: '/\.css$/',
      use: ['style-loader','css-loader']
    }]
    style-loader的作用:
    把打包后的css文件挂载到head标签中。
    css-loader的作用：
    把所有css及上下文的css依赖整合成一个css。
  ```
  + 如果使用了css预处理器的打包配置如下:
  ```
    import css from './index.scss'
    rules: [{
      test: '/\.scss$/',
      use: ['style-loader','css-loader', 'scss-loader']
    }]
    npm install sass-loader node-sass webpack --save-dev sass 依赖 node-sass
    scss-loader的作用:
    把sass语法还原成css语法
  ```
  + loader的执行顺序怎么样的？
    + 从下到上、从右到左
  + 如何处理css3的兼容前缀呢？
    + posscss-loader驾到，安装
    ```
      npm i -D postcss-loader
      rules: [{
        test: '/\.scss$/',
        use: ['style-loader','css-loader', 'scss-loader', 'postcss-loader']
      }]
    ```
    + 项目根目录下创建postcss.config.js
    ```
      npm i -D autoprefix
      module.exports = {
        require('autoprefixer')
      }
    ```
  + css-loader配置高级用法
   + 场景分析：存在index.scss中通过 @import 引入 common.scss， 如果直接通过上面的配置，那么common.scss就
     不会再走 postcss-loader 和 scss-loader。 解决方案如下：
    ```
      npm i -D postcss-loader
      rules: [{
        test: '/\.scss$/',
        use: ['style-loader', {
          loader: 'css-loader',
          options: {
            importLoaders: 2, // 解决上述问题，表示如果遇到scss中import scss文件，则重新走下方loader
            modules: true
          }
        }, 'scss-loader', 'postcss-loader']
      }]
    ```
-----
### 使用loader打包静态资源-字体文件
  ```
    rules: [{
      test: '/\.(eot|ttf|svg)$/',
      use: {
        loader: 'file-loader'
      }
    }]
  ```
---
### 什么是plugin?
  + 定义: plugin可以在webpack运行到某个时候的时候帮你做一些事情。
  + 常用plugin及功能：
    + HtmlWebpackPlugin: 自动在打包目录下生成index.html, 并把打包的js引入到index.html中
    ```
    plugins: [
      new HtmlWebpackPlugin({
        template: 模版html
      })
    ]
    ```
    + clean-webpack-plugin: 打包之前可以删除某个文件
    ```
    plugins: [
     new CleanWebpackPlugin(['dist']）
    ]
    ```
---
### entry 和 output的配置
  ```
    moudle.export = {
      entry: {
        main: './index.js',
        main2: './index.js'
      },
      output: {
        publicPath: 'http:www.cnd.cn',
        fliename: '[name].js', // 占位符name代表自动取entry配置的key值
        path: path.resolve(__dirname, 'dist')
      }
    }
  ```
---
### sourceMap配置
  + 什么是sourceMap?
    + sourceMap为一个映射关系，可以映射打包后的文件代码和源代码中的代码。
    ```
    moudle.export = {
      devtool: 'source-map'
    }
    inline-source-map: 会打包到js文件中，底部注释中
    cheap-source-map: 当我们代码量很大，代码出错， source-map会告诉你哪行哪列出错，而cheap-source-map只能知道行，性能会加速
    moudle-source-map: moudle能关联到第三方包里的错误。
    eval: 性能最好
    cheap-module-eval-source-map: 开发环境推荐配置
    cheap-moudle-souce-map: 线上如果出问题，推荐配置
    ```
---
### webpackDevServe
  + 主要用于起node服务让项目做热加载更新
    + 原始的方式可以通过在package.json 的 script 下添加 --watch属性做监听，只有webpack打包的文件发生变化就自动打包，但是需要重新刷新浏览器。
    + devServer可以解决自动刷新浏览器，并且通过参数能自动打开浏览器
    ```
      moudle.export = {
        devServe: {
          contentBase: '本地服务器路径 -> dist目录',
          open: true, // 自动打开浏览器
          proxy: {}, // 接口代理
        }
      }
    ```
    + 模拟devServer功能
    ```
    const express = require('express');
    const webpack = require('webpack');
    const webpackDevMiddleware = require('webpack-dev-middleware'); // 监听文件改变重新执行
    const config = require('./webpack.config.js');
    <!-- node中使用webpack返回编译器 -->
    const complier = webpack(config); // 编译器，相当于执行一直编译器就自动打包一次代码
    const app = express();
    app.use(webpackDevMiddleware(complier, {
      publicPath: config.output.publishPath
    }));
    app.listen(3000, () => {
      console.log('serve is running')
    })
    ```
    + 热模块更新hmr (及改了css的文件，只更新css的文件更新，只改了js的文件，只更新js的文件更新)
    ```
      moudle.export = {
        devServe: {
          hot: true, // 热加载生效
          hotOnly: true //即使热加载不生效，浏览器也不刷新
        }
      }
    ```
    + 同时配置插件 hotMoudleReplacementPlugin
    ```
      plugins:[
        new webpack.hotMoudleReplacementPlugin()
      ]
    ```
    + 如果引入一些比较偏的文件，可能需要编写hmr代码，如下：
    ```
      import moudleFile from './moudleFile'
      if(module.hot) {
        module.hot.accept('./moudleFile', () => {
          moudleFile()
        })
      }
    ```
---






