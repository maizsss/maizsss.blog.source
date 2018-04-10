---
title: koa最草根指南
date: 2018-04-10 18:11:43
tags:
---

## koa是啥

> koa——基于Node.js平台的下一代web开发框架。
> koa 是由 Express 原班人马打造的，致力于成为一个更小、更富有表现力、更健壮的 Web 框架。 使用 koa 编写 web 应用，通过组合不同的 generator，可以免除重复繁琐的回调函数嵌套， 并极大地提升错误处理的效率。koa 不在内核方法中绑定任何中间件， 它仅仅提供了一个轻量优雅的函数库，使得编写 Web 应用变得得心应手。

- 以上，是[koa官方文档](https://koa.bootcss.com/)的简介。
- 区别于[express](http://www.expressjs.com.cn/4x/api.html)，koa不提供任何中间件，只提供中间机制。
- koa目前已经发展到v2.5.0版本，要求nodejs在v7.6.0以上，对比koa1版本，全面支持es6语法，async函数。

## koa的实际使用方式
- 通过koa启动一个web服务
```
const Koa = require('koa');
const app = new Koa();

app.use(async ctx => {
  ctx.body = 'hello';
});

app.listen(3000, () => {
    console.log(' koa server start!');
});
```
- 使用中间件
```
app.use(ctx => {
  ctx.body = 'hello';
});
```
- 使用第三方中间件（路由）
```
// 使用koa-router中间件
const Koa = require('koa');
const Router = require('koa-router');
const app = new Koa();
const router = new Router();

router.get('/api', (ctx, next) => {
    console.log(`${ctx.method} ${ctx.url}`);
    console.log(ctx.router);
    ctx.body = `${ctx.method} ${ctx.url}`;
});

app
  .use(router.routes())
  .use(router.allowedMethods());

app.listen(3000, () => {
    console.log(' koa server start!');
});
```
- 在一个实际项目中的koa
```
直接看项目啊。。。
```
## koa的中间件（核心）
- 什么是中间件
    - 字面意思，就是在程序中间处理各种逻辑的套件（函数）。
    - koa-router用于处理http请求的路由逻辑，被封装成第三方中间件的形式提供出来便于开发。
    - 理解了koa的中间件机制，等于已经掌握了koa。
- [koa-compose](https://github.com/koajs/compose/blob/master/index.js)
    - 核心代码只有10行（真·核心）左右。在不支持async函数时，用Generator函数实现，支持async后改为Promise实现。
    - middleware是一个数组，里面所有元素都是函数。compose函数就是将middleware数组转化为函数套函数的调用形式。
    ```
    [g1, g2, g3] => g1(g2(g3()))
    ```
- 中间件的执行顺序
    - code
    ```
    // fn1
    app.use((ctx, next) => {
        console.log(1);
        next();
        console.log(4);
        
    });
    // fn2
    app.use((ctx, next) => {
        console.log(2);
        next();
        console.log(5);
    });
    // fn3
    app.use((ctx, next) => {
        console.log(3);
        next();
        console.log(6);
    });
    ```
    - compose展开,伪代码
    ```
    Promise.resolve(async (context, fn2) => {
    	console.log(1);
    	await Promise.resolve(async (context, fn3) => {
    		console.log(2);
    		await Promise(async (context) => {
    			console.log(3);
                // next(); 没有下一个中间件，return Promise.resolve()
                console.log(6);
    		}());
    		console.log(5);
    	})
    	console.log(4);
    }());
    ```
    - ![image](https://upload-images.jianshu.io/upload_images/3663059-03622ea2a9ffce2a.jpg?imageMogr2/auto-orient/)
    
- 如何捕获中间件的错误
    - code
    ```
    app.use((ctx, next) => {
        console.log(1);
        next();
        console.log(4);
        
    });
    app.use((ctx, next) => {
        console.log(2);
        next();
        console.log(5);
    });
    app.use((ctx, next) => {
        console.log(3);
        aaa //错误：aaa未定义
        next();
        console.log(6);
    });
    ```
    - 中间件内部的错误，只会中断本函数的运行，不会中断整个中间件流程的运行。
    - 监听error事件
    ```
    app.on('error', (err, ctx) => {
        console.error('server error', err, ctx)
    });
    ```
    - 在外层中间件捕获内层的错误
    ```
    app.use(async (ctx, next) => {
        console.log(1);
        try {
            await next();
        } catch (error) {
            console.log(error);
        }
        
        console.log(4);
        
    });
    app.use(async (ctx, next) => {
        console.log(2);
        await next();
        console.log(5);
    });
    app.use(async (ctx, next) => {
        console.log(3);
        aaa
        await next();
        console.log(6);
    });
    ```
    
- 程序解耦（为什么要用koa）
   - 不同业务逻辑之间的隔离
   - 公共处理层与业务逻辑之间的隔离
   - 提供第三方统一的接入规则，方便软件生态扩大（类比一下jq插件）
   - 如工厂流水线一样顺滑流畅的程序

## koa的功能构成
- app应用程序
    - 是一个对象
    - 包含listen、callback、use等方法
    - 包含一组中间件，context、request、response属性
- ctx上下文
    - 是一个对象
    - 包含request、response
    - 包含一些公共方法
    - 包含一些被中间件加工过后的属性
    - 每个请求都会创建一个ctx，贯通整个请求的逻辑
- req请求
    - 包含请求信息，方法
- res响应
    - 包含响应信息，方法



## 扩展
- koa与egg.js/thinkjs的关系
    - eggjs包含koa
    - 基于koa的程序组织风格，约定了一套开发规范
    - 目录结构、运行、配置、路由、任务、单元测试、日志、插件等。。。
    - 增加了一些服务端分层设计的概念，Controller、Service
    - 其实就是提供一整套开箱即用的架构代码。是大厂的脚手架搭建、开发习惯的沉淀。
- 常用的koa中间件
    - 路由：koa-router
    - body参数解析：koa-body
    - cors跨域允许：koa2-cors
    - session：koa-session
    - ...
- 如何学习koa
    - 看官方文档
    - 看项目，写项目
    - 了解原理、思路：搜索引擎
    - 进阶：参考优秀的上层框架的程序架构（如上面说到的eggjs）
