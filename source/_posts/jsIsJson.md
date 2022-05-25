---
title: JavaScript 判断字符串是否 JSON 格式
author: RealHMY
top: false
cover: false
toc: true
mathjax: false
date: 2022-05-24 15:11:16
img:
coverImg:
password:
summary:
tags:
categories:
---

有用户反馈，页面上的数据不显示，查看后台数据，数据正常，但页面上的数据无法正常显示。打开控制台，看到有报错信息，看代码逻辑应该是处理json时报错了，仔细查看，发现领导写出了这样的代码：

```js
<div v-if="data.content && data.content.indexOf('[') !== -1">
```

判断json格式居然是根据有无方括号来判断的？？？

人都麻了，这里贴一下判断字符串是否json格式的代码：

```js
isJSON(str) {
    if (typeof str == 'string') {
        try {
            let obj = JSON.parse(str);
            return !!(typeof obj == 'object' && obj);
        } catch (e) {
            console.log(e);
            return false;
        }
    }
    return false;
}
```

哪怕是领导写的代码也要谨慎使用，谨记！！！