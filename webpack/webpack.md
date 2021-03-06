# 一、webpack基本格式

#### 核心配置

entry（入口文件路径），output（打包输出路径），module（loader），plugin（plugins），mode（开发模式）五个模块

执行打包命令：webpack

```javascript
//引入绝对路径拼接方法
const {resolve} = require('path');

//webpack是以nodejs为运行环境，因此采用commonjs的模块使用方式
module.exports = {
	//打包文件路径（入口文件路径）: 同级文件夹路径下的src中的index
	entry:'./src/index.js',
	//输出文件及路径 : 输出到创建好的build文件夹并创建文件（一般为build.js）
	output:{
		filename:'build.js',
		path:resolve(__dirname,'build')
	}
	//loader
	module:{
		rules:[
			{
				test:/\.css$/,//匹配文件类型
				use:[
					'style-loader',//创建style标签，将样式放入
					'css-loader'//将CSS文件整合到js中
				]//顺序是从后向前执行
			}
		]
	}
	//plugins
	Plugin:[
        new HtmlWebpackPlugin({
            template:'复制html文件路径'
        })
    ],
	//开发模式
	mode:'development',
    //自动打包工具，需要下载webpack-dev-server
    devServer:{
        static:resolve(__dirname,'build'),//运行文件路径
        compress:true,//gzip压缩
        port:3000,//本地3000端口运行
        open:true//是否自动运行
    }
}
```

# 二、基础使用

#### loader的使用：1.下载 2.使用

```javascript
module:{
    rules:[
        /*处理css文件*/
        {
            test:/\.css$/,
            use:['style-loader','css-loader']
        },
        /*处理less文件*/
        {
            test:/\.css$/,
            use:['style-loader','css-loader','less-loader']
        },
        /*处理css中的图片资源*/
        {
            test: /\.(png|jpg|gif)$/i,
            use: [
                   {
                     loader: 'url-loader',
                     options: {
                         limit: 8192,
                         esModule: false,
                         name:'[hash:5].[ext]'
                     },
                   },
             ],
             type: 'javascript/auto'
        },
        /*处理html文件中img标签中的图片*/
        //必须把处理css图片中的url-loader的esModule关了
        {
            test:/\.html$/,
            loader:'html-withimg-loader'
        },
        /*打包其他资源*/
        {
            excludes:/.\(css|js|html)$/,
            loader:'file-loader'
        }
    ]
}
```

#### plugins的使用:1.下载2.引入3.使用

```javascript
/*处理html*/
const HtmlWebpackPlugin = require('html-webpack-plugin')
plugins:[
	//下载模块html-webpack-plugin
    //默认创建一个空的html，自动引入打包输出所有资源（JS,CSS...）
	new HtmlWebpackPlugin({
        //复制'./src/index.js'文件，并自动引入打包输出所有资源
    	template:'./src/index.js'
    })
]
```

# 三、devServer的使用

下载：webpack-dev-server

本地运行命令：npx webpack  server

```javascript
devServer:{
    static:resolve(__dirname,'build'),//运行文件绝对路径
    compress:true,//是否采用gzip压缩
    port:3000,//本地运行端口号
    open:true//是否自动运行打开
}
```

# 四、CSS文件相关

## 提取CSS成单独文件

下载：mini-css-extract-plugin

```javascript
const MiniCssExtractPlugin = require('mini-css-extract-plugin')
module:{
    rules:[
        {
            text:/\.css$/,
            use:[
                MiniCssExtractPlugin.loader,
                'css-loader'
            ]
        }
    ]
}
plugins:[
	new MiniCssExtractPlugin({
        //单独创建css文件到
    	filename:'css/build.css'
    })
]
```

## CSS兼容性处理

需要使用：postcss-loader   postcss-preset-env

```javascript
{
  loader: "postcss-loader",
  options: {
    postcssOptions: {
      plugins: [
        [
          "postcss-preset-env",
          {
            // Options查询githnb
          },
        ],
      ],
    },
  },
 },
```

针对不同浏览器的版本兼容：在package.json配置`browserslist`

## 压缩CSS

需要使用： [css-minimizer-webpack-plugin](https://github.com/webpack-contrib/css-minimizer-webpack-plugin) 

 :exclamation: 需要与MiniEXtractCssPlugin配合使用

```javascript
/*生产环境*/
const MiniCssExtractPlugin = require("mini-css-extract-plugin");
const CssMinimizerPlugin = require("css-minimizer-webpack-plugin");

module.exports = {
  module: {
    rules: [
      {
        test: /.s?css$/,
        use: [MiniCssExtractPlugin.loader, "css-loader", "sass-loader"],
      },
    ],
  },
  optimization: {
    minimizer: [
      // For webpack@5 you can use the `...` syntax to extend existing minimizers (i.e. `terser-webpack-plugin`), uncomment the next line
      // `...`,
      new CssMinimizerPlugin(),
    ],
  },
  plugins: [new MiniCssExtractPlugin()],
};
/*开发环境*/
module.exports = {
  optimization: {
    // [...]
    minimize: true,
  },
};
```



# 五、开发环境配置

```javascript
const HtmlWebpackPlugin = require('html-webpack-plugin');
const {resolve} = require('path');
module.exports = {
    entry:'./src/index.js',
    output:{
        filename:'build.js',
        path:resolve(__dirname, 'build')
    },
    module:{
        rules:[
            {
                test:/\.css$/,
                use:[
                    'style-loader',
                    'css-loader'
                ]
            },
            {
                test:/\.less$/,
                use:[
                    'style-loader',
                    'css-loader',
                    'less-loader'
                ]
            },
            {
                test:/\.(png|jpg|gif)$/,
                loader:'url-loader',
                options:{
                    limit: 8*1024,
                    name:'[hash:5].[ext]',
                    esModule:false
                }
            },
            {
                test:/\.html$/,
                loader:'html-withimg-loader'
            },
            {
                exclude:/\.(html|css|less|png|jpg|gif)/,
                loader:'file-loader'
            }
        ]
    },
    plugins:[
        new HtmlWebpackPlugin({
            template: './src/index.html'
        })
    ],
    mode: 'development',
    devServer: {
        static:resolve(__dirname,'build'),//编译文件夹
        compress:true,
        open: true,
        port:3000
    }
}
```

# 六、性能优化

## 开发环境性能优化：

### 方向1：提升webpack打包效率

<font color="pink">HMR（hot module replacement）</font>：当一个模块发生更新时，只更新这一个模块

```javascript
devServer:{
    contentBase:resolve(__dirname,'built'),
    compress:true,
    port:3000,
    open:true,
    hot:true//开启HMR
}
```

------

css文件：默认支持HMR功能；

js文件：默认不支持；

```javascript
//在入口JS文件中，按需进行HMR
if(module.hot){
    //监听需要启动HMR的js模块，一旦发生变化启动回调
    module.hot.accept('path',function(){
        js();//执行模块的功能
    })
}
```

html文件：默认不支持同时热更新失效；

```javascript
//解决热更新失效问题：修改入口文件
entry:['./src/js/index.js','./src/index.html']
```

------

### 方向2：优化代码调试

<font color="pink">source-map</font>：提供能够映射源代码和构建后代码映射关系的功能

```javascript
//直接在webpack.config.js文件中插入
devtool:'source-map'
```

分为：[inline- | hidden- | eval-] [nosources- ] [cheap- | cheap-module- ] source-map 这几种

其中<font color="pink">hidden-映射到构建后的文件</font>；<font color="pink">nosources-找不到映射文件</font>;内联映射文件是inline和eval

**使用场景**

1. ​	开发环境：速度（eval > inline > hidden）和调试友好程度(cheap-module)

   ​	eval-source-map  ||  eval-cheap-module-source-map  

2. ​    生产环境：安全和调试友好程度，不考虑内联

   ​    source-map

## 生产环境性能优化：

### 方向1：提升webpack打包效率

### 方向2：优化代码运行性能