---
title: buffer
date: 2019-10-18 16:14:16
tags: 开发
categories: node.js 
---
1. 在引入TypeArray之前，JavaScript语言没有用于读取或操作的二进制数据流的机制。Buffer类是作为Node.jsAPI的一部分引入的，用于在TCP流、文件系统操作、以及其他上下文中与八位字节流进行交互。[Node.js官网对Buffer的描述](http://nodejs.cn/api/buffer.html)
2. Buffer代表一个数据缓存区，用于存储二进制数据，俗称字节流，是I/O传输时常用的处理方式。相较于字符串，Buffer可以免去编码解码的过程，节省CPU成本。

## 二进制
在计算机的世界里，只有0和1，也就是二进制。
任何计算机可以识别或者存储的数据都是二进制。
![二进制](https://ss0.bdstatic.com/70cFuHSh_Q1YnxGkpoWK1HF6hhy/it/u=1337836810,3632387218&fm=26&gp=0.jpg)
## Stream
1. stream不是Node.js独有的概念，而是操作系统的最基本方式，Node.js有API支持这种方式
2. stream是对buffer对象的高级封装，其底层还是buffer。
3. 流是一个抽象的数据接口，是EventEmitter对象的一个实例，以Buffer为单位。
4. http的请求、响应，stdut（标准输出）等。
![流的形象表示](https://www.runoob.com/wp-content/uploads/2015/09/bVcla61)
## Buffer的使用场景
1. 网络数据
在使用net或http模块来接收`网络数据`时，可用buffer作为数据结构进行传输，即`data`事件的参数
接收微信服务器响应的代码片段，接收所有的buffer转换未string
Buffer与字符串间转换会产生极小的性能损耗
```
   if (sha === signature) {
      let buf = '';
      req.setEncoding('utf8');
      req.on('data', (chunk) => {
          buf += chunk; // 拼接的过程中有隐私的转换，相当于调用toString()方法
      });
      req.on('end', async () => {
        const content = await this.appService.xmlToJson(buf);
        this.appService.sendMsg(content, query.openid);
        res.status(HttpStatus.OK).json({body: 'success'});
      });
    } else {
      throw new HttpException('Forbidden', HttpStatus.FORBIDDEN);
    }
```

2. 大文件
用于`大文件`的读取和写入，以前fs读取的内容是string，后来都改用Buffer，在大文件读取上，性能和内存上有明显的又是。
```bash
const fs = require('fs')
const inputStream = fs.createReadStream('input.txt')
const outputStream = fs.createWriteStream('output.txt')
inputStream.pipe(outputStream) // 管道读写
```

3. 转码、进制转换
4. 数据结构
用作`数据结构`，处理二进制数据，也可以处理字符编码

### Buffer的创建
```
new Buffer() // 废弃的API，不推荐

Buffer.from()
Buffer.alloc()
Buffer.allocUnsafe() //泄漏内存，不安全
Buffer.allocUnsafeSlow() // 泄漏内存，不安全
```
### 字符串与Buffer的相互转换
1. 字符串转Buffer
Buffer.from()不传递第二个参数会按照UTF-8格式进行转换
```bash
Buffer.from('Node.js', 'UTF-8')
```

2. Buffer转字符串
Node.js中有一个字符串解码`类`,即string_decoder(字符串解码器)，StringDecoder可以将缓存Buffer解码未字符串，StringDecoder是buffer.toString()方法的简单实现。
```bash
declare module "string_decoder" {
    class StringDecoder {
        constructor(encoding?: string);
        write(buffer: Buffer): string;
        end(buffer?: Buffer): string;
    }
}
```

```bash
const { StringDecoder } = require('string_decoder')

const stringDecoder = new StringDecoder()
const buf = Buffer.from('Node.js', 'UTF-8')
stringDecoder.write(buf)

buf.toString()
```
## Buffer（缓冲）VS Cache（缓存）
1. 缓存（Buffer）是用于处理二进制流数据，将数据缓冲起来，它是临时性的，对于流数据，会采用缓冲区将数据临时存储起来，等缓冲到一定的大小之后再存入硬盘中。视频播放就是一个经典的例子，有时你会看到一个缓冲的图标，这意味着此时这一组缓冲区并未填满，当数据到达填满缓冲区并且被处理之后，此时缓冲图标消失，你可以看到一些图像数据
2. 缓存（Cache）我可以看作是一个中间层，他可以额是永久性的将热点数据进行缓存，使得访问速度更快，例如我们通过Memore、Redis等将数据从硬盘或其他第三间方接口中请求过来进行缓存，目的就是将数据于内存的缓存区内，这样对同一个资源进行访问，速度会更快，也是性能优化的一个重要点。浏览器中的缓存目的也是为了不用重复去加载相同的资源，进行的优化措施。