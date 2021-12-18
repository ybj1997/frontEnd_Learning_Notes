## CSR与SSR的区别

CSR的例子：通过React脚手架生成的项目中， body中除了兼容处理的noscript标签之外，只有一个id为root的标签。  首页的内容是  下面的script中拉取的JS代码控制的。 

​	CSR（客户端渲染）与SSR（服务端渲染）最大的区别在于前者的页面渲染是JS负责的，而后者是服务器端直接返回HTML让浏览器端渲染。

### 传统CSR弊端

- 由于页面显示过程要进行JS文件拉取和React代码执行，首屏加载时间会比较慢。
-  对于SEO 不友好， 为搜索引擎爬虫只认识html结构的内容，而不能识别JS代码内容。 

## 简单实现React组件的服务端渲染

 JSX中的标签其实是基于虚拟DOM的，最终要通过一定的方法将其转换为真实DOM， 而react-dom这个库中刚好有实现了编译虚拟DOM的方法。

```jsx
//component
import React from 'react';
const Home = () => {
  return (
    <div>
      <div>This is sanyuan</div>
    </div>
  )
}
export default Home
```

利用react-dom库中编译VDOM的方法

```jsx
import express from 'express';
import { renderToString } from 'react-dom/server';
import Home from './containers/Home';

const app = express();
const content = renderToString(<Home />);//编译VDOM
app.get('/', function (req, res) {
   res.send(
   `
    <html>
      <head>
        <title>ssr</title>
      </head>
      <body>
        <div id="root">${content}</div>
      </body>
    </html>
   `
   );
})
app.listen(3001, () => {
  console.log('listen:3001')
})
```

## 同构

​	由于服务器端没有DOM事件绑定，因此在上一节中，给基础函数组件绑定事件是无效。因此我们需要对编译代码进行同构。

### 什么是同构？

 	通俗的讲，就是一套React代码在服务器上运行一遍，到达浏览器又运行一遍。服务端**渲染完成页面结构**，在上一节已经完成，接下来让浏览器端渲染**完成事件绑定**。 

####  浏览器端的事件绑定 

  	那如何进行浏览器端的事件绑定呢？  唯一的方式就是 让浏览器去拉取JS文件执行，让JS代码来控制。 

