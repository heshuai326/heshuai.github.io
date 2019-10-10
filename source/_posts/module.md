---
title: module
date: 2019-10-10 16:33:56
tags: 开发
categories: node.js 
---
>java有类文件，Python有import机制，Ruby有require等，而Javascript 通过script标签引入代码的机制显得杂乱无章，语言自身毫无组织能力，人们不得不用命名空间的等方式人为的组织代码，以求达到安全易用的目的
《深入浅出Nodejs》--朴灵
# CommonJs
## 模块化的作用
1. 代码的复工和工程解耦
2. 全局环境下的变量命名冲突
3. 解析依赖，管理依赖

## node的模块实现
- 路径分析
- 文件定位
- 编译执行
![模块加载机制](https://raw.githubusercontent.com/Qbian61/Qbian61.github.io/master/resource/nodejs-module/nodejs-require.jpg)
1. 核心模块：fs、http等，就像jdk的核心类
核心模块在编译过程中编译成了二进制文件。在进程启动时核心模块就被加载进内存中，所以后面2个步骤就可以省略，并且在路径分析中优先级最高（除了缓存区）
2. 文件模块：用户编写模块

## 模块编译
在node中每个js文件都被是为独立的模块
- js文件：通过模块`同步`读取文件后编译执行
- json文件：通过fs模块`同步`读取文件后，用JSON.parse()解析返回结果
- node文件：这是用C/C++编写的扩展文件，通过dlopen()方法加载最后编译生成的文件

1.每个编译成功的模块都会将其文件路径做为索引缓存在Module.cache对象上，以提高二次引用的性能
2.在编译的过程中，node对获取的js文件内容进行了封装
```bash
(function(export, require, module, __filename, __dirname){
// 模块代码实际在这里
})
```
这样在模块之间就进行了作用域的隔离，保持了顶层的变量在模块范围内，而不是全局对象
提供了一些看似全局但实际上是模块特定的变量
实现者可以用鱼从模块中导出值得module和exports对象
包含模块绝对文件名和目录路径的快捷变量__filename和__dirname
# ES6模块
在ES6之前，社区指定了一些模块方案CommonJs（服务端）AMD（浏览器），ES6模块成为浏览器和服务器通用的模块方案
## ES6模块和CommonJs模块的差异
- CommonJs模块输出的是一个值拷贝，ES6模块输出的是值引用
- CommonJs模块是运行时加载，ES6模块是编译时的输出接口

```bash
let firstName = 'Michael'
let lastName = 'Jackson'

let year = 1958

// 导出变量
export { firstName, lastName, year}


// 导出函数或类(Class)
export function multiply(x, y) {
    return x * y
}

// 为了给用户方便，为模块指定默认输出
export default function fn() {

}
```
```bash
/**
 * commonjs规范 (服务端)
 * AMD         （浏览器）
 */ 
// const { stat, exists, readFile } = require('fs')

// ES6 在编译时就完成模块编译，效率要比Common模块高
// const {stat, exists, readFile } from 'fs'


//使用按时关键字将输入的变量重命名
import { lastName as surname}  from './export'

// default的时候不需要写{}
import fn from './export'

import * as obj  from './export'
```