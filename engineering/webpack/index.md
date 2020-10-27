# webpack

Webpack 打包原理和基本配置

loader，它是一个转换器，将 A 文件进行编译成 B 文件，比如：将 A.less 转换为 A.css，单纯的文件转换过程。

plugin 是一个扩展器，它丰富了 webpack 本身，针对是 loader 结束后，webpack 打包的整个过程，它并不直接操作文件，而是基于事件机制工作，会监听 webpack 打包过程中的某些节点，执行广泛的任务

Webpack 处理项目时，它会递归地构建一个依赖关系图，其中包含应用程序所需要的每个模块，然后将这些模块打包成一个或者多个 bundle。
打包原理： 1.识别入口文件 2.通过逐层识别模块依赖（commonjs、amd、es6 module），webpack 都会进行分析，来获取代码的依赖关系。
3.webpack 做的就是分析代码、转换代码、编译代码、输出代码 4.最终形成打包后的代码。

基本配置：
￼
entry: 入口文件 output：在哪里输出 bundledevtool：开发者工具 source-map
devServer：创建开发服务器 plugins:[ new CleanWebpackPlugin([‘dist’]), //删除 dist 目录,
new HtmlWebpackPlugin({title:””})// 生成 html 文件] ,
Mode:’development’,
Module:{ rules:[ test:/\.css\$/.
use:[‘style-loader,’css-loader’] ]}

- 注意，对于 loader 的执行顺序，是从后往前的。

项目常用配置：

￼

预加载文件：sass-resources-loader，定义的公共的 scss 文件，为了不再每个页面都引入，设置文件预加载。

￼

常用 loaders
Less-loader/sass-loader/postcss-loaderurl-loader file-loader// 小于 8k 的图片自动转成 base64 格式，并且不会存在实体图片,两个都必须用上。否则超过大小限制的图片无法生成到目标文件夹中
babel-loader，babel-preset-es2015，babel-preset-react js 处理，转码
eslint-loader

Tree-shaking ：Tree-shaking 的本质是消除无用的 js 代码，实际情况中，虽然依赖了某个模块，但其实只使用其中的某些功能。通过 tree-shaking，将没有使用的模块摇掉，这样来达到删除无用代码的目的。

找到你整个代码里真正使用的代码，打包进去，那么没用的代码自然就剔除了。tree shaking 得以实现，是依赖是 es6 的模块特性。
关于 es6 module 的特性，大概有如下几点：

1. 必须写在最外层，不能写在函数里
2. import 的语句具有和 var 一样的提升(hoist)特性。
   tree shaking 首先会分析文件项目里具体哪些代码被引入了，哪些没有引入，然后将真正引入的代码打包进去，最后没有使用到的代码自然就不会存在了。

常用 plugin:
html-webpack-plugin:多入口时，当你的  index.html  引入多个 js，如果这些生成的 js 名称构成有  [hash] ，那么每次打包后的文件名都是变化的。
imagemin-webpack-plugin:图片过大，加载速度慢，浪费存储空间。
clean-webpack-plugin:每次进行打包需要手动清空目标文件夹。
CommonsChunkPlugin:提取被重复引入的文件，单独生成一个或多个文件，这样避免在多入口重复打包文件。
copy-webpack-plugin:一些静态资源（图片、字体等），在编译时，需要拷贝到输出文件夹。
DllPlugin 拆分 bundles，加载构建速度

webpack 常用的 loader

- 样式：style-loader、css-loader、less-loader、sass-loader 等
- 文件：raw-loader、file-loader 、url-loader 等
- 编译：babel-loader、coffee-loader 、ts-loader 等
- 校验测试：mocha-loader、jshint-loader 、eslint-loader 等
