---
title: "我，想加个号"
tags:
  - DataCrawl
---

由于某些原因，需要增加个某动的手机号。手机号毕竟是会跟着自己很长一段时间的东西，所以还是希望好好挑选一下的。
但进到某动的官网后，这么简单而粗暴的号码搜索过滤功能，顿时无爱了。

<figure class="align-center">
  <img src="{{ site.url }}{{ site.baseurl }}/assets/images/posts/10086.search.png" alt="">
  <figcaption>某动号码搜索.</figcaption>
</figure>

{% include toc %}

根本就没有一些高端的搜索功能，比如，还有`1392736108y`这样的`3^x`这样的连号吗？还有`13612481632`这样的以`2^x`这样的整数号吗？
（#啊啊啊，臭码农！！！）还有`136xyzyx631`这样的回文号吗？

所以，为了能够做一些更~~蛋疼~~有深度的号码查找，我们得去拿到现在某动库里的所有号码啊。好吧，那就干吧！

~~打开某动的号码页面，然后拷贝这一页中的所有号码放到一个文件中，保存一下；然后点击下一页，然后再拷贝这页中的号码到文件中，然后...喂喂喂，某动有10776个号码，有539页呢，为了办个号我手都要残废了喂.... (抓狂中...)~~

## 号码抓取

现在爬数据这种技术其实已经还是比较成熟了（#我只是装作好像知道现状似的）,像[scrapy](https://scrapy.org/)不仅使用方便而且功能强大。但就为了抓手机号码这种事情，去写个基于scrapy的爬虫，总有种用牛刀的感觉（其实主要是因为我不会用scrapy）。现在的浏览器，不管[Chrome](https://developers.google.com/web/tools/chrome-devtools/console/)、[Firefox](https://developer.mozilla.org/en-US/docs/Tools/Web_Console)、[Safari](https://developer.apple.com/library/content/documentation/AppleApplications/Conceptual/Safari_Developer_Guide/Console/Console.html)等等都有控制台的功能。在控制台里，用户可以通过`Javascript`对当前的页面进行各种各样交互式的操作。
所以现在的任务就是如何利用Console，将令人抓狂的手动复制粘贴翻页这个过程用程序来完成。

在我们进入Console之后（Chrome/Firefox 按`F12`可以打开），在Console的最下面有个闪烁的光标可以进行输入。例如，我们可以尝试`document`，这个时候Console会把整个网页的document对象返回；例如，`document.querySelectorAll('tr')`将会把网页中，所有的表格行返回。有了这个功能以后，我们只要知道需要获取网页中那部分的内容，就可以通过Console的这些内容选取功能获得数据了。

### 获取当前网页中的手机号码

<figure style="width: 400px" class="align-right">
  <img src="{{ site.url }}{{ site.baseurl }}/assets/images/posts/console.inspect.png" alt="">
  <figcaption>Console inspect.</figcaption>
</figure>

为了获取手机号码，所以，我们首先需要知道手机号码在页面中是被怎样展现的。在Console的左上角有一个带箭头的Inspect Selector按钮，点击这个按钮后，再将鼠标移动到网页中的某个元素上点击后，Console就会跳转到Elements中，并且告诉我们网页中被点击的那个元素在网页代码中对应的内容位置。有了这个功能之后，我们就可以查看网页中的手机号码都是被存放在什么地方的。
<figure class="align-center">
  <img src="{{ site.url }}{{ site.baseurl }}/assets/images/posts/console.element.png" alt="">
  <figcaption>号码在网页源代码中的位置.</figcaption>
</figure>

<figure style="width: 400px" class="align-right">
  <img src="{{ site.url }}{{ site.baseurl }}/assets/images/posts/10086.trs.png" alt="">
  <figcaption>网页中的所有号码所在的所有tr元素.</figcaption>
</figure>

通过对网页源码的观察，我们可以发现，网页中所有的号码都是被放置在`id`为`mainNum`的`div`中的`tr`里。有了这个发现，我们就可以通过在Console里执行Javascript来获得号码内容了。

获取所有号码元素的代码如下：

``` javascript
var mainNum = document.querySelector('#mainNum');
var trs = mainNum.querySelectorAll('tr');
console.log(trs);
```

接下来，就是将这些号码逐个解析出来。
从图中我们发现，某动其实也将号码作为每个号码对应的`id`属性，所以我们只需要获取每个`<tr>`的`id`，然后把号码从这个id字符串中解析出来即可。

``` javascript
var numbers = new Set();
for (var tr of trs) {
  var id = tr.getAttribute('id');
  if (id !==null && id !== undefined) {
    var fields = id.split('|');
    if (fields.length === 6 && fields[1].length === 11) {
      numbers.add(fields[1]);
    }
  }
}
console.log(numbers);
```

这样我们就将当前页面下的所有号码都获取到并且保存在`numbers`集合中了。

### 获取其他页面中的号码

只能获取当前页面并保存的话，脚本的意义就没有了。所以我们还需要让脚本能够自动对号码内容进行翻页。
想要达到这个目标，我们需要让脚本能够在跳转到页面的输入框中输入相应的页面编号，然后让脚本自动的点击`Go`按钮。
所以，我们首先需要找到这两个元素的网页代码。
同样使用上面提到的Inspect功能，我们发现页面中`到  页`和`Go`按钮对应的网页元素分别是：

``` javascript
<input type="text" id="kkpager_btn_go_input" value="">
```

以及

``` javascript
<div id="kkpager_btn_go" onclick="jump(538)">Go</div>
```

因此，我们可以通过以下代码获得这两个元素。

``` javascript
var btnGoInput = document.querySelector('#kkpager_btn_go_input');
var btnGo = document.querySelector('#kkpager_btn_go').onclick();
```

然后计算下一页页面值，并在`input`中填入相应内容，并执行跳转。
（其实也可以通过模拟点击`下一页`达到目的，但某动的网页中`下一页`这个元素获取起来比较别扭，所以我就通过计算下一页页面值来进行跳转了）

``` javascript
var currPageNum = parseInt(document.querySelector('.currPageNum').innerText);
var nextPageNum = currPageNum + 1;
btnGoInput.setAttribute('value', nextPageNum);
btnGo.onclick();
```

这样，页面在获取完当前页面后，就可以进行下一个页面中号码的获取了。

### 让代码循环执行

在能跳转了以后，我们还需要让代码能够自动的在下一页执行号码获取操作，并且在下一页完成后，继续在下下一页中获取号码。这个时候，我们就可以使用javascript的`setInterval`函数了。这个函数的功能是间隔一定时间执行一遍指定的回调函数。

``` javascript
var refreshIntervalId;
var totalPageNum = parseInt(document.querySelector('.totalPageNum').innerText);

function getNumbersInCurrentPage() {
  // 获取当前所有号码
  // ...

  if (nextPageNum > totalPageNum) {
    if (refreshIntervalId !== null && refreshIntervalId !== undefined) {
      clearInterval(refreshIntervalId);
    }
  } else {
    // 设置下一页页面序号，并进行跳转
    document.querySelector('#kkpager_btn_go_input').setAttribute('value', currPageNum + 1);
    document.querySelector('#kkpager_btn_go').onclick();
  }

  // ...
}

refreshIntervalId = setInterval(getNumbersInCurrentPage, 5000);
```

完整的获取号码的代码可以查看我的[Github](https://github.com/huajianmao/userscripts/blob/master/src/10086.js)

这样，你只要把代码拷贝到Console里执行，并且在网络条件还可以，并且不被某动禁IP的情况下等待`totalPageNum * 5 / 60`分钟后得到结果了。
当然，也可以在获取每一页号码后，将号码存到localStorage中，这样可以防止页面意外退出而白等了脚本执行的时间。
完整代码如下（无localStorage）：

``` javascript
  var refreshIntervalId;
  var numbers = new Set();
  var totalPageNum = parseInt(document.querySelector('.totalPageNum').innerText);

  function getNumbersInPage() {
    var mainNum = document.querySelector('#mainNum');
    var currPageNum = parseInt(document.querySelector('.currPageNum').innerText);
    console.log("Going to get numbers in " + currPageNum);

    var trs = mainNum.querySelectorAll('tr');
    for (var tr of trs) {
      var id = tr.getAttribute('id');
      if (id !==null && id !== undefined) {
        var fields = id.split('|');
        if (fields.length === 6 && fields[1].length === 11) {
          numbers.add(fields[1]);
        }
      }
    }

    var gotoPageNum = currPageNum + 1;
    if (gotoPageNum > totalPageNum) {
      if (refreshIntervalId !== null && refreshIntervalId !== undefined) {
        clearInterval(refreshIntervalId);
      }
    } else {
      document.querySelector('#kkpager_btn_go_input').setAttribute('value', currPageNum + 1);
      document.querySelector('#kkpager_btn_go').onclick();
    }

    console.log("Have fetched " + numbers.size + " numbers!");
  }

  refreshIntervalId = setInterval(getNumbersInPage, 5000);
```

## 号码查找

有了号码以后，能做的事情就看你想要什么号码了。
然后，写代码对numbers集合进行处理，到这里就完全是个人喜好了。


``` javascript
// 获取所有回文号码

```

``` javascript
// 获取包含一定pattern的号码

```

``` javascript
// ...
```

## 后记
这篇文章其实是在我已经挑了一个号码后才写的，所以并没有完整的让它抓取所有的号码，不知道真是抓到后面的时候会有怎样的情况发生。
如果这篇文章有任何侵犯到您的权利，请联系我删除。
