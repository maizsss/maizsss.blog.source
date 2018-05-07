---
title: 从nodejs的stream说起
date: 2018-05-07 09:17:23
tags:
---

> 周末，实在无聊。看看人，看看车，还不如回家看兔子。看兔子也看不久，喂几个零食后不再喂，兔子就该拉的拉该发呆的继续发呆。还是看点技术的东西吧。

## 前言
- 好啦，这个文章就这么开头了。
- 我进入互联网行业是从web前端开始的，其实直到现在也是从事着前端开发相关的职位，只不过工作性质已经不单指是前端了，还包括了产品策划（渣渣水平）、服务器运维（也是渣渣）、nodejs开发。web前端开发已经很热了（其实热了好几年了，估计还差不多热过头了），前端的热起在技术上其实真的离不开nodejs。很多人以为nodejs只是个能让javascript跑在服务端环境的一个平台而已，然后能做的事情与PHP这些是类似的，都是提供一个webserver。其实也说得没错，的确是这样。但程序员的脑洞很大，也非常地懒惰，他们除了拿nodejs来做webserver之外，还拿来做前端的工程化工具、各种奇奇怪怪新兴平台的脚本语言。尤其是前端工程化，这让web前端的逼格越了好几个档次。实现各种流程的自动化、前后端分离、跨平台的编译工具，硬生生地把前端的职责范围撑大了（我该感谢还是抱怨）。还是感谢吧，毕竟nodejs赐予了我能接触web开发全貌的机会。
- nodejs中文网首页最显眼的是这么一段
```
Node.js 是一个基于 Chrome V8 引擎的 JavaScript 运行环境。 
Node.js 使用了一个事件驱动、非阻塞式 I/O 的模型，使其轻量又高效。 
Node.js 的包管理器 npm，是全球最大的开源库生态系统。
```
- 我尝试逐句解释。
    1. 这句好理解，关键词“V8”、“运行环境”，nodejs是个环境而非语言，nodejs使用V8来运行javascript的。
    2. 这句比较难理解，我先查了一下其他文章。说是事件驱动，其实是因为nodejs的多数模块其实是继承了事件模块（events），I/O就是“对资源的读和写”，非阻塞就是在程序上的“读写”操作不阻塞后面程序的运行。至于为什么说是轻量和高效，我就是不知道了，其实很多语言都说自己是轻量高效啊。
    3. npm，分开来读单词：nodejs package manager，对javascript的开源包无论服务端还是浏览器端还是其他端都适用（所以没可能不大嘛）。
- 以上都是为了凑篇幅，下面正题吧。在平日nodejs的使用上，多数会通过上层库或框架去使用，以致对底层的原理了解很少。这次尝试去学习一个重要的模块stream去理解nodejs。

## nodejs的stream
- [stream（流）](http://nodejs.cn/api/stream.html)是nodejs很重要的一个模块。在很多场景都用到了流，比如：前端工程化、网络、大数据量的处理。
- 在目前的前端开发里，因为每个人都对gulp很熟悉了，gulp是前端进入工程化的一个重要的里程碑工具，配合上各种插件能帮助前端解决掉大量代码压缩、预编译等等工序。不认识的话看一段代码。
```
// 这是一段less的预处理脚本
function styles() {
  return gulp.src(paths.styles.src)
    .pipe(less())
    .pipe(cleanCSS())
    // pass in options to the stream
    .pipe(rename({
      basename: 'main',
      suffix: '.min'
    }))
    .pipe(gulp.dest(paths.styles.dest));
}
```
- 上面代码看到了几个gulp的方法：“src、pipe、dest”，另外的还有watch、task 。这些在翻开[gulp的github仓库](https://github.com/gulpjs/gulp/blob/master/index.js)就能清晰了，然而gulp的核心不在这里，这里只是对各个模块的封装和暴露api的入口而已。再看看[gulp的组](https://github.com/gulpjs),重要的源码仓库是在“vinyl-fs”、“glob-watcher”、“to-through”、“glob-stream”这些，这里面会引用到“[through2](https://github.com/rvagg/through2/blob/master/through2.js)”,这其实就是对nodejs的stream模块的再次封装。不再细说gulp，这只是引出stream在工程化上的使用场景而已。换句话说，想了解nodejs怎么引用在工程化上面，读gulp源码是个好选择。
- stream有四种类型：
    - Readable - 可读的流 (例如 fs.createReadStream()).
    - Writable - 可写的流 (例如 fs.createWriteStream()).
    - Duplex - 可读写的流 (例如 net.Socket).
    - Transform - 在读写过程中可以修改和变换数据的 Duplex 流 (例如 zlib.createDeflate()).
- stream.pipe()是官方推荐的消费流数据的主流方式。
```
// 创建可读流，读取文件数据
const r = fs.createReadStream('file.txt');
// zlib是nodejs用于压缩的模块，也是继承于stream。
const z = zlib.createGzip();
// 创建可写流
const w = fs.createWriteStream('file.txt.gz');
// 读取文件，数据提供给消费者z，z再把数据提供给消费者w
r.pipe(z).pipe(w);
```
- 以上所说到的数据，其实都是操作Buffer，缓冲器。顾名思义就是流传输中的暂存区域。
![image](https://mmc-forecast.oss-cn-shanghai.aliyuncs.com/images/44a8d7fe733ce1-544x420.png)

## nodejs的其他模块与stream的关系
- 上面举到了gulp这个在前端工程化实际使用流的例子。其实nodejs其他模块（重要模块）都有很多流的影子，而且基本涵盖常用模块。
- http/https
```
const http = require('http');

const server = http.createServer((req, res) => {
  // req 是 http.IncomingMessage 的实例，这是一个 Readable Stream
  // res 是 http.ServerResponse 的实例，这是一个 Writable Stream

  let body = '';
  // 接收数据为 utf8 字符串，
  // 如果没有设置字符编码，将接收到 Buffer 对象。
  req.setEncoding('utf8');

  // 如果监听了 'data' 事件，Readable streams 触发 'data' 事件 
  req.on('data', (chunk) => {
    body += chunk;
  });

  // end 事件表明整个 body 都接收完毕了 
  req.on('end', () => {
    try {
      const data = JSON.parse(body);
      // 发送一些信息给用户
      res.write(typeof data);
      res.end();
    } catch (er) {
      // json 数据解析失败 
      res.statusCode = 400;
      return res.end(`error: ${er.message}`);
    }
  });
});

server.listen(1337);
```
- net.socket就是 Duplex 的实例，它的可读端可以消费从套接字（socket）中接收的数据， 可写端则可以将数据写入到套接字。 简单看看api吧。
```
new net.Socket([options])
创建一个 socket 对象。

options <Object> 可用选项有：
    fd: <number> 如果指定了该参数，则使用一个给定的文件描述符包装一个已存在的 socket，否则将创建一个新的 socket。
    allowHalfOpen <boolean> 指示是否允许半打开的 TCP 连接。详情查看 net.createServer() 和 'end' 事件。默认是 false。
    readable <boolean> 当传递了 fd 时允许读取 socket，否则忽略。默认 false。
    writable <boolean> 当传递了 fd 时允许写入 socket，否则忽略。默认 false。
Returns: <net.Socket>
```
- fs模块里也有使用流,上文也有类似的代码例子。
```
fs.ReadStream 类
fs.WriteStream 类
fs.createReadStream(path[, options])
fs.createWriteStream(path[, options])
```
- crypto加密模块
```
// 使用Cipher和管道流:

const crypto = require('crypto');
const fs = require('fs');
const cipher = crypto.createCipher('aes192', 'a password');

const input = fs.createReadStream('test.js');
const output = fs.createWriteStream('test.enc');

input.pipe(cipher).pipe(output);
```

## nodejs的events
> “所有的流都是 EventEmitter 的实例。”

> “大多数 Node.js 核心 API 都采用惯用的异步事件驱动架构，其中某些类型的对象（触发器）会周期性地触发命名事件来调用函数对象（监听器）。”
- 以上是官方原话，印证了"事件驱动"的这一个概念。
- events实例的api很简洁，eventEmitter.on() 方法用于注册监听器，eventEmitter.emit() 方法用于触发事件。其余都是“辅助”性质的api。
- “事件模型”在浏览器端已经很常见了。vue也提供on、emit这些api。学名就叫做“订阅发布”。
```
const EventEmitter = require('events');

class MyEmitter extends EventEmitter {}

const myEmitter = new MyEmitter();
myEmitter.on('event', () => {
  console.log('触发了一个事件！');
});
myEmitter.emit('event');
```

## 总结
- 文章主要介绍了stream、events两个模块，目的不是深入这些模块的原理、用法，平日的使用中也极少几率直接用到，都是间接地在使用。我是希望通过翻找出nodejs的“基石”，联系各个模块的关系，从而帮助自己能构建出nodejs的全貌。
- 而就是这么简洁明快的模块，nodejs在这之上构建出完整的一套api，支持着开发者在众多复杂场景下面的使用。
- 有错误望指出，谢谢。
