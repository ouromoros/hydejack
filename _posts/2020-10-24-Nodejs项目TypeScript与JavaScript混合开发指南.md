---
layout: post
title: 'Node.js项目TypeScript与JavaScript混合开发指南'
tags: [Other]
categories: [Chinese]
---

Node.js作为服务器开发框架来说性能足够达到要求，有各种丰富的库支持，开发速度也快，很适合后台业务服务的开发。但JavaScript作为一个编程语言各方面实在是惨不忍睹，涉及到复杂的业务逻辑时代码复杂度会迅速变大，非常难以维护，因此使用微软为解决这个问题所创建的TypeScript才是正解，但罗马不是一日建成的，主观上我们希望原来的项目可以马上切换到TypeScript，但实际上我们不免要面对遗留代码以及老的JS库的问题，这就导致。

关于TypeScript如何与JavaScript一起工作，网上实际上已经有不少相关的文档了，官方文档中专门有一篇文章介绍[如何从JavaScript项目迁移TypeScript](https://www.typescriptlang.org/docs/handbook/migrating-from-javascript.html)。可惜的是这在很多时候并不能解决我们的需求，因为实际开发时很多时候我们没有时间也没有意愿（懒得）去修改以前已经写好的代码，而只希望可以在旧项目的基础上增添新的逻辑。

在项目中同时存在`.ts`文件与`.js`文件时，我们需要掌握这些文件互相引用的方法，否则就无法好好开发。

## 开始之前

首先需要在项目中安装`typescript`以及`@types/node`，`typescript`就不用说了，`@types/node`中则帮我们定义好了`require`、`exports`等node代码中使用的方法和变量的类型，否则我们在`.ts`文件中使用`require`等node的方法时会报错误。

```console
$ npm i -D typescript @types/node
```

然后为了让`tsc`编译时将`.js`文件也包含进去（其实就是把它们原封不动的复制到构建目标文件夹），我们需要在`tsconfig.json`中加上`allowJS=true`，这也是在官方文档中所提到的。

## 如何引用第三方库

对于支持TypeScript的库一般安装之后就可以放心使用，导入时使用`import * as`的语法就可以导入原来使用`require`所获取的对象。

如果库本身不支持类型（提示找不到库的错误）可以尝试安装`@types/<package_name>`，一般可以解决问题。如果没有`@types`库的话就只能自己编写`.t.ds`文件补充库的类型（详情可见[这篇文章](https://www.detroitlabs.com/blog/2018/02/28/adding-custom-type-definitions-to-a-third-party-library/)），或者使用原生的`require`方法导入。

当库缺少类型时我推荐使用`require`方法直接导入，这样一方面是省心，一方面是可以提醒你这个库是导入的第三方没有类型文件的库，在使用时需要更加小心。

## 如何在JavaScript中引用TypeScript文件

在编写项目时我们可能只想用TypeScript编写新的一部分代码，那么在编写完之后势必需要在旧的JavaScript代码中调用新编写的逻辑，网上对这部分并没有详细的介绍，因此在这里简单进行介绍。

在要被引用的`.ts`文件中，开发时我们不需要注意什么，只要在最后使用Node.js中JavaScript原生的方式`module.exports`的方式进行导出，这行语句在编译之后会原封不动的留下来，因此可以被其它`.js`文件引用。需要注意的是原生的方式和TypeScript中的导出方式不能混用，否则编译之后会发生冲突，无法被正确导入。

在`.js`文件中我们只需要正常的使用`require`方法导入对应的模块即可。

## 如何在TypeScript中引用JavaScript文件

对原来的`.js`文件不需要进行更改，我们只需要关心如何在`.ts`文件中导入它们。

网上对于如何做到这点一共有三种方法，如下：

1. 使用`import * as js_module form './js_module'`
2. 使用`import js_module = './js_module'`
3. 使用`const js_module = require('js_module')`

在经过实践之后我认为第三种方法是最好的方法，即使用`require`导入JavaScript模块。一方面这个语法可以将JavaScript模块和其它TypeScript模块的导入区分开，另一方面这也是最可靠的方式，因为它在编译后依然是相同的正确的语句，而其它方法有时会因为TypeScript版本或是其它的配置原因发生错误。
