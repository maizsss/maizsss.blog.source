---
title: headlesschrome封装库puppeteer的运用
date: 2018-06-11 14:27:47
tags:
---
## 介绍一下puppeteer
- [官方说明](https://github.com/GoogleChrome/puppeteer)
> Puppeteer is a Node library which provides a high-level API to control headless Chrome or Chromium over the DevTools Protocol. It can also be configured to use full (non-headless) Chrome or Chromium.
- 我练习一下翻译
> Puppeteer是一个提供了高级api（高层封装）可以操作headless Chrome和基于Chromium的开发者工具协议的nodejs库。它能被设置为无头模式的chrome和chromium。
- 通俗点说的就是，我们可以用Puppeteer这个nodejs库来操作一个浏览器（chrome）。
- 在Puppeteer之前，我们如果需要做一些自动化浏览器的事情，很容易会想到PhantomJS，或者nightmare。然而听说了Puppeteer出来之后，PhantomJS直接放弃维护了（真假不清楚，听说的）。而之前一直在用nightmare的开发者，听说过Puppeteer之后，也换阵营了（比如说本人，还有ant design pro的UI e2e测试方案，详见[#1006的更新日志](https://pro.ant.design/docs/changelog-cn)）。
- 在粗略翻了一下官方的[api文档](https://github.com/GoogleChrome/puppeteer/blob/v1.4.0/docs/api.md)之后，就能发现Puppeteer的功能之广，涵盖到“跳转页面、点击、dom元素检查、js脚本注入”的基本操作，也有“截图”、“PDF”、“性能”、“请求、响应”、“移动端事件”等能想到的浏览器功能。PhantomJS能做到的一些爬虫功能，Puppeteer能轻松应对，而且提供了更多的其余功能，更高的运行性能。而nightmare因为基于electron，electron本身也个强大的框架，所以nightmare在其之上还算是有一定的抵抗力。对比背后团队，puppeteer是GoogleChrome支撑开源的，项目维护实力无需担心。
## 使用方式
### 基本用法
- npm安装
```
npm i puppeteer
```
- hello world
```
/* example.js */

const puppeteer = require('puppeteer'); // 引入puppeteer

(async () => {
  const browser = await puppeteer.launch();// 实例化一个浏览器对象
  const page = await browser.newPage();// 调起一个新的页面
  await page.goto('https://example.com');// 跳转到一个url
  await page.screenshot({path: 'example.png'});// 对当前页面进行截图

  await browser.close();// 关闭一个浏览器对象
})();
```
- 运行
```
node example.js
```
- 以上是官网上的代码例子，更多的不再说，请详细翻阅api文档和其他的使用例子。但从这几行代码可以看出，puppeteer的api从理解上和使用上都十分地友好，基本上遵循了我们平时对chrome的使用习惯。
- 另外，可以清楚的是，puppeteer运行在nodejs平台，作为一个npm包去引入，与npm强大的生态资源配合起来使用，可以做很多事情。
### linux环境下使用
- 一般来说，我们会在windows或者mac系统下使用chrome。而linux，对于像chrome这样的GUI应用程序来说并不合适在其之上运行。然而还是有方式。
- 当npm安装puppeteer时，会默认安装了chromium。但安装了chromium并不意味着马上就可以在linux上运行，因为还缺少一些依赖。
- 以centos7为例，需要执行以下命令安装一系列依赖。
- 安装这些依赖成功后，就可以让puppeteer以headless模式运行了。
- 除此之外，当你对一些页面进行截图时会发现，有些汉字显示不完整，这是因为缺少了中文字体库和一些字库依赖。那就安装一下。
- 至此，puppeteer就能运行在linux下了。
## 使用场景
- 在官网上，已经列出了几条puppeteer的主要使用方式。
    - Generate screenshots and PDFs of pages.
Crawl a SPA and generate pre-rendered content (i.e. "SSR").
    - Automate form submission, UI testing, keyboard input, etc.
    - Create an up-to-date, automated testing environment. Run your tests directly in the latest version of Chrome using the latest JavaScript and browser features.
    - Capture a timeline trace of your site to help diagnose performance issues.
- 我在实际使用中，也参考了这几点去实践了。下面大概说说。
### 作为ui自动化测试的浏览器框架
- 在UI自动化测试这个事情上面，之前我有总结过一个博文：[点击跳转](https://maizsss.github.io/2017/10/28/%E5%89%8D%E7%AB%AF%E7%9A%84UI%E8%87%AA%E5%8A%A8%E5%8C%96%E6%B5%8B%E8%AF%95/)。所以不再细说。
- 这次我把以上博文中的nightmare换成了puppeteer，也就是只把自动化浏览器框架调换了，其他技术构成基本保持一致。
- 此外，我还利用这套UI自动化测试去做了“页面监控”这样的事情。这里不再展开，有兴趣可以私下讨论。
### 自动网页截图发送
- 以上代码例子中就有用到了screenshot这个api，网页截图就是基于这个api开展。仔细看[screenshot的文档](https://github.com/GoogleChrome/puppeteer/blob/v1.4.0/docs/api.md#pagescreenshotoptions),还能发现它支持“全屏”、“图片类型、质量指定”、“裁剪”。
- 要做成这个功能，需要用到nodejs的其他一些工具库，比如说“定时任务”、“邮件发送（或其他消息通知类工具）”、“打包压缩”。是一个puppeteer很好的工具配合使用的实践。

### SPA项目的爬虫
- SPA（单页面web应用）在页面渲染上面用的都是js异步渲染，与传统服务端渲染直接吐出完整的html内容不同。所以一些旧的爬虫工具已经不太适合使用在SPA项目上面了。
- 而puppeteer因为是一个chromium内核的浏览器工具，固然能抓取到所有能在页面正常渲染的内容，无论是html还是基于js的异步加载。
- puppeteer本身支持单个浏览器对象的多开页面，所以在共享登录态这类场景下毫无压力。视硬件配置而定，甚至能多开浏览器实例，做并发爬虫。

### 服务端渲染
- 也是因为puppeteer能抓取到所有能在页面正常渲染的内容，换个方向去想，其实也能在抓取这些内容后生成完整的html内容。
- 猜想而已。一个SPA项目，在webserver层判断当下请求是否需要一个完整的html内容（比如百度爬虫），是的话就通过puppeteer抓取对应页面的内容后生成html返回（这里可以适当做一下缓存）。不是的话，就返回SPA页面。

## 一点思考
- 其实不止nightmare、puppeteer，另外还有一个叫[chromeless](https://github.com/prismagraphql/chromeless)的库，也是在做着类似的事情。也是基于headless chrome的库。我也花了时间去做调研对比。
- 这是当下这个前端时期的精彩和焦虑。客户端配置越来越高，让js的性能发挥得越来越好，js的跨平台特性，让它可以在各个领域都能开花，这是精彩。而焦虑的是，各种层出不穷的技术快速映入开发者的视野，作为js使用者我们不能避而不看。而又有很多的技术框架其实是功能类似的，我们要如何在这当中选择最合适的来使用，很可能会让我们要在这中间来回跳转好几次。
- 利弊都有，而在我内心还是十分感激js开源社区中活跃的前辈们赋予了我们很多的工具，很多的能力，让我们能做更多的事情，改善工作，改善生活。
