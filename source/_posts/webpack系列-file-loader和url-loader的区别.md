---
title: 'webpack系列: file-loader和url-loader的区别'
date: 2018-04-17 23:15:56
categories:
  - Javascript
tags:
  - Javascript
  - webpack
---

## 前言
在使用webpack的时候，需要引入各类Loader，但是很多Loader的工作原理其实不是很清楚，多数都是网上教程上让加上去的。这篇文章希望能搞懂file-loader和url-loader深层次一些的东西。

## 什么是Loader
Loader可以简单的理解为模块和资源的转换器。我们知道webpack是模块管理工具，在webpack处理应用程序时，他会将所有资源分成各类模块来引入，比如js, css, image等等。由于模块的类型大不相同，有时候我们需要进行对这些模块进行预处理，这时候就需要Loader来干活了。

由于本文探讨的是file-loader和url-loader的区别，Loader的一些基本概念就不提了，具体内容请自行查看[*官方介绍*](https://webpack.js.org/concepts/loaders/)

## file-loader 和 url-loader的作用
在开发过程中，当我们需要引入图片时，会遇到一些问题。

### 引用路径的问题
通过background引入背景图片，在我们通过webpack打包后，所有的资源打包在bundle.js里，此时样式中的url路径是相对index.html而不是相对于最初的原始文件路径，这样就会导致图片引入失败。

这个问题用file-loader可以解决，file-loader可以解析项目中的url引入，根据我们的配置，将图片拷贝到相应路径，再根据我们的配置，修改打包后文件引用路径，使之指向正确的位置。

### 图片较多带来的问题
当图片较多时，会发起很多的HTTP请求，这样会造成页面性能下降。这个问题可以通过url-loader解决。url-loader会将引入的图片编码，生成DataURL。然后将DataURL打包进js文件中。当然如果文件较大，编码会消耗性能。因此，url-loader提供了一个limit配置，小于limit字节的文件会被转换成DataURL，大于limit的会通过file-loader进行处理。

<!-- more -->

## file-loader和url-loader的关系
虽然url-loader可以通过file-loader的方式去处理文件，但是url-loader不依赖file-loader。因为url-loader自身内置了file-loader。所以使用url-loader只需安装url-loader而不需要安装file-loader。

url-loader的工作方式分两种：

- 文件小于limit的情况，将文件转换为DataURL

- 文件大于limit的情况，通过file-loader进行处理

## 使用方式

```
module.exports = {
  module: {
    rules: [
      {
        test: /\.(png|jpg|gif)$/,
        use: [
          {
            loader: 'url-loader',
            options: {
              limit: 1024
            }
          }
        ]
      },
      {
        test: /\.(png|jpg|gif)$/,
        use: [
          {
            loader: 'file-loader',
            options: {}
          }
        ]
      }
    ]
  }
}
```

## 总结
由于url-loader可以通过limit来区别对待不同大小的文件，同时内置了file-loader，所以我们只需要单独安装url-loader就可以了。

> 文章内容参考资料:
> 
> [webpack官网 - file-loader](https://doc.webpack-china.org/loaders/file-loader/)
> 
> [webpack官网 - url-loader](https://doc.webpack-china.org/loaders/url-loader/)
> 
> [webpack学习笔记 - file-loader 和 url-loader](https://blog.csdn.net/qq_38652603/article/details/73835153)

