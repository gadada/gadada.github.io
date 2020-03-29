---
title: "不买房，就看看"
tags:
  - DataCrawl
---

房价已经是疯了，不知道还会怎样疯。

在[我，加个号]({% post_url 2017-02-12-crawl-phone-numbers %})这篇文章里，介绍了一下如何利用Console来获取某动上的可用手机号码列表。趁着还有点热情，顺便就把抓某家在售二手房的简单信息的方法给写一下吧。

爬某家的房价信息原理和抓某动的信息是类似的，就是获取页面中指定的一些元素的信息并保存，然后跳转到下一页中，进行下一页信息的抓取。但某家的翻页和某动的翻页有些不一样，某家的翻页是真的页面变换，而某动的翻页是数据内容的翻页而不是页面之间的跳转。所以在某家下没法直接通过在Console里设定循环翻页的方式来进行数据抓取。（因为翻页后，Console里的脚本就已经被清除了。）所以在抓取房价数据的时候，我们得想一个另外的办法来循环获取数据。这里我们先介绍一下[Tampermonkey](https://tampermonkey.net/)这个工具。

## Tampermonkey

 > Tampermonkey is a free browser extension and the most popular userscript manager. It's available for Chrome, Microsoft Edge, Safari, Opera Next, and Firefox. 
 >
 > Even though some of the supported browsers have native userscript support, Tampermonkey will give you much more convenience in managing your userscripts. It provides features like easy script installation, automatic update checks, a simple overview what scripts are running at a tab, a built-in editor and there is a good chance that incompatible scripts run fine with Tampermonkey. 
 >
 > Tampermonkey is available for Google Chrome, Microsoft Edge, Opera, Chromium, a lot of their derivatives like CoolNovo and Rockmelt and also Firefox and some Android browsers. It's installed in just a minute, so give it a try! 

Tampermonkey提供了许多很有意思的功能。用户可以对指定的URL页面做一些指定的操作，比如添加一些界面元素、修改一些界面样式、对页面中的内容进行操作等等。只要用户查看的地址，能够被指定URL pattern匹配上，那么Tampermonkey就会自动的去执行预先设定的Javascript代码。这刚好能够满足我们的需求，当用户打开浏览器并进入到某家的房价列表页面的时候，自动去执行房价信息解析的代码，并在获取该页面信息后，自动的去执行页面跳转。当页面跳转后，因为跳转后的URL依然能够被Tampermonkey捕获，所以它会继续执行脚本操作，并周而复始的抓取所有的数据。

## 关键元素提取

通过查看网页源码，我们可以发现所有的房源信息都是被放在以`clear`类的`<tr>`里的，我们可以通过`querySelectorAll`来获取房源列表：

``` javascript
var houseLis = document.querySelectorAll('li.clear');
```

有了这个房源列表后，我们只要再对其中相应的内容进行提取就能获得列表中每个房源的所有信息了。

``` javascript
for (var house of houseLis) {
    var houseObj = {};
    houseObj.id = "";
    houseObj.xiaoqu = {};
    houseObj.district = {};
    houseObj.price = {};
    houseObj.tags = [];

    var image = house.getElementsByTagName('img')[0];
    houseObj.avatar = image.getAttribute('src');

    var title = house.getElementsByClassName('title')[0].getElementsByTagName('a')[0];
    houseObj.title = title.text;
    houseObj.url = title.getAttribute('href');
    if (houseObj.url !== undefined && houseObj.url !== null) {
        var lastSlashIndex = houseObj.url.lastIndexOf("/");
        if (lastSlashIndex >= 0) {
            var subString = houseObj.url.substring(lastSlashIndex + 1);
            var endIndex = subString.indexOf(".html");
            houseObj.id = subString.substring(0, endIndex);
        }
    }

    var houseInfo = house.getElementsByClassName('address')[0].getElementsByClassName('houseInfo')[0];
    var xiaoqu = houseInfo.getElementsByTagName('a')[0];
    houseObj.xiaoqu.name = xiaoqu.text;
    houseObj.xiaoqu.url = xiaoqu.getAttribute('href');
    houseObj.info = houseInfo.innerText;

    var positionInfo = house.getElementsByClassName('flood')[0].getElementsByClassName('positionInfo')[0];
    var district = positionInfo.getElementsByTagName('a')[0];
    houseObj.district.name = district.text;
    houseObj.district.url = district.getAttribute('href');
    houseObj.position = positionInfo.innerText;

    houseObj.followInfo = house.getElementsByClassName('followInfo')[0].innerText;

    var tagSpans = house.getElementsByClassName('tag')[0].getElementsByTagName('span');
    for (var tagSpan of tagSpans) {
        houseObj.tags.push(tagSpan.innerText);
    }

    var priceInfo = house.getElementsByClassName('priceInfo')[0];
    houseObj.price.total = priceInfo.getElementsByClassName('totalPrice')[0].innerText;
    houseObj.price.unit = priceInfo.getElementsByClassName('unitPrice')[0].getAttribute('data-price');

    houses[houseObj.id] = houseObj;
}
```

## 数据本地保存

因为还涉及到不同页面中数据的汇总，如果仅是用一个数据结构存放在内存中的话，在页面跳转后数据将会被清空，所以我们需要让脚本在获取每个页面的内容后对数据进行保存。

`localStorage`是HTML5标准提供的对数据在本地进行存储的一种方式。
它以`key-value`的形式对数据进行存储，用户可以通过`localStorage.get('KEY')`获取数据以及`localStorage.set('KEY', 'VALUE')`存储数据。
基于此，我们可以把房源信息列表序列化成json格式，然后存放到`localStorage`中。

``` javascript
var houses = localStorage.getItem('houses');
if (houses === null || houses === undefined) {
    houses = {};
} else {
    houses = JSON.parse(houses);
}

// 从网页中获取所有数据
// ...

localStorage.setItem('houses', JSON.stringify(houses));
```

## 跳转到下一页

页面中可以进入到下一页的方法其实有很多，人工合成URL、点击`下一页`等。这里采用下一页的方式来进行页面跳转。
首先观察网页代码，发现`下一页`按钮是被放置在一个`house-lst-page-box`类的`<div>`中的最后一个`<a>`中的。

``` javascript
setTimeout(5000);
window.scrollTo(0, document.body.scrollHeight);

var pagenation = document.querySelector('.house-lst-page-box');
var pageData = JSON.parse(pagenation.getAttribute('page-data'));
var totalPage = pageData.totalPage;
var curPage = pageData.curPage;

if (curPage < totalPage) {
    var links = pagenation.querySelectorAll('a');
    var nextPageLink = links[links.length - 1];
    if (nextPageLink.innerText === "下一页") {
        nextPageLink.click();
    }
}
```

## 让Tampermonkey自动执行
到此为止，我们已经有了抓取页面中房源信息的脚本了，剩下的就是保证当我们打开某家的房源列表页面时，这个脚本能够被正确的执行。
这是Tampermonkey正擅长的事情。我们通过点击Tampermonkey插件，并选择`Create a new script...`来新建一个脚本即可。脚本内容如下：

``` javascript
// ==UserScript==
// @name         MoujiaSaver
// @namespace    http://hjmao.me/
// @version      0.1
// @description  To save moujia brief info in local storage!
// @author       Huajian Mao
// @include      http://hz.moujia.com/ershoufang/*
// @grant        none
// ==/UserScript==

(function() {
    'use strict';

    // 信息获取，并存储到localStorage中
    // ...
})();
```

脚本中比较重要的地方时是`@include`，它指明了URL的pattern，当我们打开某个网页时，如果该网页是以`http://hz.moujia.com/ershoufang/`为开头的，本脚本就会在该页面被执行。

完整代码详见[Github](https://github.com/huajianmao/userscripts/blob/master/src/lianjiaSaver.js)。

## 后记
在等待漫长的时间后，所有获取的数据都会被存放在`Developer tools` -> `Application` -> `Local Storage` -> `http://hz.moujia.com`中的houses这个变量中，你可以双击对应的`value`列然后把里面的值拷贝出来，或者直接在Console里通过`localStorage.get('houses')`来获取内容，然后再做相应的后续操作。

代码中有许多隐含bug的地方，比如某个对象没找到的时候，获取的数组为空的时候，等等，都没有做检查。
如果恰巧你碰到了，自行添加检测吧。

如果这篇文章有任何侵犯到您的权利，请联系我删除。
