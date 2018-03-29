---
layout: blog
title: "react 安装"
background-image: http://121.43.158.69/img/2.jpg
date:  2018-03-15
category: react
tags:
- react
---

##安装 react 注意 如果安装不成功 很多都是目录权限问题 mac 请执行 sudo  -s
````
安装React方法
React是灵活js库并能够支持多种不同的项目。可以直接使用它创建新的项目，同时支持在已有的项目下引入。

尝试React
如果仅仅对react感兴趣，可以直接使用CodePen. 就可以不需要安装任何东西，同时只需要修改代码和观看结果。尝试可以点击这里。

如果想使用编辑器编辑，可以下载HTML文件。修改后，在浏览器上展示即可。但是请不要把这个使用到正式产品，因为这里使用的react是网络下载的，会比较慢。

如果想学习使用完整的应用。可以看以下内容：创建React项目或者在已有项目中引入React。

创建React项目
创建新的React项目，从一个简单的页面学起是最好的学习方法。这需要设置开发环境，并使用最新的js方法、提供最新的开发体验，和优化你的项目。

npm install -g create-react-app    //安装环境
create-react-app my-app    //创建应用 应用名称my-app 最好使用正确的路径

cd my-app    //移动项目
npm start  //开始使用
React项目不需要处理后台逻辑和数据库，仅仅是用于前端。如果使用Balel和webpack，就不再需要配置其他内容。

当想要把项目部署在正式环境，使用npm run build就可以创建一个优化过的项目到build文件夹中。想要了解更多的创建项目请看ReadMe和UserGuide。

添加React到已有项目
不需要因为引入React而重新开项目
推荐先在项目中部门内容使用React，如individual widget。

React可以不在构建工具下使用，推荐自定义设置项目，这样的效率更佳，一般情况下有以下设置内容

package manager，包（库）管理。如Yarn或者npm。更容易管理，安装和升级第三方包，
bundler，构建器。如：wabpack或者browserify。更加容易的把模块合并到一起，并且优化加载时间。
compiler，转换器。如：Babel。更好的支持旧版本的js语法。
安装React
安装须知：强烈推荐设置production build process确保在项目使用的最新的React。

推荐使用Yarn或者npm来管理前后台依赖。如果是初次接触包管理器，推荐使用 Yarn documentation。

//安装Yarn方法
yarn init
yarn add react react-dom
//安装npm方法
npm init
npm install --save react react-dom
无论使用Yarn或者npm下载资源，都来源于npm registry。

使用ES6和JSX
推荐使用Babel让ES6和JSX运行在JS代码里。ES6拥有一些列最新的JS特性，使得开发更简单。JSX是js语言扩展，更好的应用在react。

Babel setup instructions 解释了如何配置不同的构建环境。确保项目中安装了babel-preset-react
和 babel-preset-es2015
。文件如： .babelrc
configuration。

使用ES6和JSX的Hello World
推荐使用wabpack或者browserify。更加容易的把模块合并到一起，并且优化加载时间。

//最少的React代码运行hello world
import React from 'react';
import ReactDOM from 'react-dom';

ReactDOM.render(
  <h1>Hello, world!</h1>,
  document.getElementById('root')
);

//代码中使用了一个ID叫root的div。
//所以html的代码中必须要有`<div id="root"></div>`
同样，可以在已有的项目中使用JS UI 库。

作者：ZMJun
链接：https://www.jianshu.com/p/907baa220dfd
來源：简书
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。
````
