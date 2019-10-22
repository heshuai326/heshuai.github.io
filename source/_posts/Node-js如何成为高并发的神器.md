---
title: Node.js如何成为高并发的神器
date: 2019-10-21 10:01:17
tags: 开发
categories: node.js 
---
Javascript是一种单线程的，不像Java等其他多线程可以创建多个线程并行执行。在面对高并发的场景下，Node.js那不是很脆弱么？
## 单线程
1. 事件轮询（事件驱动）和异步I/O（非阻塞I/O）
在事件驱动模型中，每个I/O工作被添加到事件队列中，线程循环地处理队列上得工作任务，当执行过程中遇到阻塞（读取文件、查询数据库）时，线程不会停下来等待结果，而是留下一个处理结果得回调函数，转而继续执行队列中的下一个任务。这个传递到队列中的回调函数在阻塞任务运行结束后才被线程调用。
![](https://timgsa.baidu.com/timg?image&quality=80&size=b9999_10000&sec=1571633472182&di=e54516920fb269a9d2b2721836ce394c&imgtype=0&src=http%3A%2F%2Fimg0.coin163.com%2F48%2F09%2Fby2mie.png)
2. Javascript是单线程的但是不管是浏览器还是Node.js环境都是多线程的。
Node.js在接受任务的时候是多线程的，无需切换进程/线程，非常高效，但它在执行具体任务的时候是多线程的。
Libuv由事件循环和线程池组成，负责所有的I/O任务的分发与执行。
![](https://images2018.cnblogs.com/blog/1136599/201805/1136599-20180524145348812-64839149.png)
3. Nginx和Apache一样，是一个HTTP服务器，它可以完胜Apache不是用的带有阻塞的多线程方式，而是带有异步I/O的事件轮询，它因此变成了响应能力更强的解决方案。

## 异常处理
## 子进程
### process
process对象是一个全局变量，它提供有关当前Node.js进程的信息并对其进行控制，无须引用，在任意位置都可以使用。
1. 统计信息：CPU、内存
2. 事件循环：nextTick
3. 异常捕获：uncaughtException
4. 其他：进程、I/O、路径

### 子进程的实现 
1. 主进程
```bash
const child_process = require('child_process')
const child = child_process.fork('./child.js')
child.on('message', function(m){
    console.log('来自子进程的消息', m)
})
child.send({ from: '父进程消息'})
```

2. 子进程
子进程是一个独立的V8实例，可以在进程中做一些耗时操作，然后在通知主进程
```bash
process.on('message', function(m){
    console.log('来自父进程的消息', m)
})
const exec = require('child_process').exec
exec('ls process.js', function(err, stdout, stderr){
    console.log(stdout)
    console.log(stderr)
    process.send({stdout:stdout, stderr: stderr })
})
```

## 集群
单个Node.js实例运行在单个线程中。为了充分利用多个系统，又是需要启动一组Node.js进程去处理负载任务。
### cluster模块
Node.js提供了集群模块，简单讲就是复制一些可以共享TCP连接的工作线程。
工作进程由child_process.fork()方法创建，因此它们可以使用IPC和父进程通信，从而使各进程交替处理连接服务。

```bash
const cluster = require('cluster');
const http = require('http');
const numCPUs = require('os').cpus().length;

if (cluster.isMaster) {
  console.log(`主进程 ${process.pid} 正在运行`);

  // 衍生工作进程。
  for (let i = 0; i < numCPUs; i++) {
    cluster.fork();
  }

  cluster.on('exit', (worker, code, signal) => {
    console.log(`工作进程 ${worker.process.pid} 已退出`);
  });
} else {
  // 工作进程可以共享任何 TCP 连接。
  // 在本例子中，共享的是 HTTP 服务器。
  http.createServer((req, res) => {
    res.writeHead(200);
    res.end('你好世界\n');
  }).listen(8000);

  console.log(`工作进程 ${process.pid} 已启动`);
}
```

### cluster的两种分发连接方法
1. 循环法，由主进程负责监听端口，接收新连接后再将连接循环分发给工作进程，再分发中使用了一些内置技巧防止工作进程任务过载。
2. 主进程创建监听socket后发送给感兴趣的工作进程，由工作进程负责直接接收连接。

### PM2
1. 集群模式启动
```bash
pm2 start app.js -i 4
```
-i参数告诉pm2以cluster_mode的形式运行你的app（对应的叫fork_mode），后面的数字表示要启动的工作线程的数量。如果给定的数字为0，pm2则会根据你的CPU核心的数量来生成对应的工作线程。
2. 实时扩展集群
```bash
pm2 scale app +3
```
需要增加工作线程的数量，可以同pm2来对集群进程扩展
## 多线程
### thread
Node.js由于Javascript执行在单一线程，导致CPU密集计算的任务可能会使主线程处于繁忙的状态，进而影响服务的性能，虽然可以通过child_process模块创建子进程的方式来解决，但是一方面进程之间无法共享内存，另一方面创建进程的开销也不小。
```bash
const {
    isMainThread, parentPort, workerData, threadId,
    MessageChannel, MessagePort, Worker
} = require('worker_threads')

function mainThread() {
    const worker = new Worker(__filename, { workerData: 0 })

    worker.on('exit', code => {
        console.log(`main：工作线程退出 ${code}`)
    })
    worker.on('message', msg => {
        console.log(`main: receive ${msg}`)
        worker.postMessage(msg + 1)
    })
}

function workerThread() {
    console.log(`worker: threadId ${threadId} 启动 ${__filename}`)
    console.log(`worker: workerDate ${workerData} `)
    parentPort.on('message', msg => {
        console.log(`worder: 收到${msg}`)
        if (msg === 5) {
            process.exit()
        }
        parentPort.postMessage(msg)
    })
    parentPort.postMessage(workerData)

}

if (isMainThread) {
    mainThread()
} else {
    workerThread()
}
```
### 真正的多线程——[node-threads-a-gogo](https://github.com/xk/node-threads-a-gogo)

## 数据库

### Redis

### 其他数据库
### I/O密集型和CPU密集型