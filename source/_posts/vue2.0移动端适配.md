---
title: vue2.0移动端适配vw整理
date: 2018-11-05 20:30:10
tags: vue
---

大漠老师的文章[查看](https://www.w3cplus.com/mobile/vw-layout-in-vue.html)


## 使用vue-cli新建项目

``` 
vue init webpack vue-demo
cd vue-demo
npm run dev
```
## 安装依赖
``` 
npm i postcss-aspect-ratio-mini postcss-px-to-viewport postcss-write-svg postcss-cssnext postcss-viewport-units cssnano cssnano-preset-advanced --S
```
## postcssrc.js配置
``` js
module.exports = {
  "plugins": {
    "postcss-import": {},
    "postcss-url": {},
    "postcss-aspect-ratio-mini": {},
    "postcss-write-svg": { utf8: false },
    "postcss-cssnext": {},
    "postcss-px-to-viewport": {
      viewportWidth: 750,
      unitPrecision: 3,
      viewportUnit: 'vw',
      selectorBlackList: ['.ignore', '.hairlines'],
      minPixelValue: 1,
      mediaQuery: false
    },
    "postcss-viewport-units": {},
    "cssnano": {
      preset: "advanced",
      autoprefixer: false,
      "postcss-zindex": false
    }
  }
}

```

### 各插件的功能：

``` js
"postcss-px-to-viewport": { 
    viewportWidth: 750, // 视窗的宽度，对应的是我们设计稿的宽度，一般是750 
    viewportHeight: 1334, // 视窗的高度，根据750设备的宽度来指定，一般指定1334，也可以不配置 
    unitPrecision: 3, // 指定`px`转换为视窗单位值的小数位数（很多时候无法整除） 
    viewportUnit: 'vw', // 指定需要转换成的视窗单位，建议使用vw 
    selectorBlackList: ['.ignore', '.hairlines'], // 指定不转换为视窗单位的类，可以自定义，可以无限添加,建议定义一至两个通用的类名 
    minPixelValue: 1, // 小于或等于`1px`不转换为视窗单位，你也可以设置为你想要的值 
    mediaQuery: false // 允许在媒体查询中转换`px` 
}
```

[postcss-px-to-viewport文档](https://link.jianshu.com/?t=%27https%3A%2F%2Fgithub.com%2Fevrone%2Fpostcss-px-to-viewport%27)

postcss-write-svg 实现Retina屏1像素边框
首先记得在heade头加入
``` html
<meta name="viewport" content="width=device-width,initial-scale=1,maximum-scale=1,minimum-scale=1,user-scalable=no" />
```
#### 最重要的 降级处理
使用 [Viewport Units Buggyfill](https://link.jianshu.com/?t=https%3A%2F%2Fgithub.com%2Frodneyrehm%2Fviewport-units-buggyfill) 插件

``` html
<script src="//g.alicdn.com/fdilab/lib3rd/viewport-units-buggyfill/0.6.2/??viewport-units-buggyfill.hacks.min.js,viewport-units-buggyfill.min.js"></script>
```

在Index.html文件body标签后添加以下代码

``` js
<script>
  // vw兼容性处理viewport-units-buggyfill
    window.onload = function () {
      window.viewportUnitsBuggyfill.init({ hacks: window.viewportUnitsBuggyfillHacks });
      //以下代码用户测试
      // var winDPI = window.devicePixelRatio;
      // var uAgent = window.navigator.userAgent;
      // var screenHeight = window.screen.height;
      // var screenWidth = window.screen.width;
      // var winWidth = window.innerWidth;
      // var winHeight = window.innerHeight;
      // console.log("Windows DPI:" + winDPI + ";\ruAgent:" + uAgent + ";\rScreen Width:" +
      //   screenWidth + ";\rScreen Height:" + screenHeight + ";\rWindow Width:" + winWidth +
      //   ";\rWindow Height:" + winHeight)
    }
  </script>
```
最后做个对img兼容处理，在全局添加(在main.js 用 Import '@/common/index.css')
``` css
img {
  content: normal !important;
}
```

如果不想用css来处理img兼容，可以使用过滤将其过滤掉具体在 postcssrc.js 中配置规则

``` js
"postcss-viewport-units": {
  filterRule: rule =>
    rule.selector.indexOf("::after") === -1 &&
    rule.selector.indexOf("::before") === -1 &&
    rule.selector.indexOf(":after") === -1 &&
    rule.selector.indexOf(":before") === -1 &&
    rule.selector.indexOf("img") === -1
},

//为了防止给 伪类 和 图片增加 content ，
```

同时要使用scss需要安装 node-scss 和 sass-loader

```
npm install sass-loader node-sass --save-dev
```