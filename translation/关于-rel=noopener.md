[英文原版](https://mathiasbynens.github.io/rel-noopener/)

## 1. 它在解决什么问题

你当前正在查看 `index.html`  

想象下以下是用户在你的网站上生成的内容：  

```html
<a class="user-generated" href="malicious.html" target="_blank">
    <b>Click me!!1 (同域)</b>
</a>
```

点击以上的链接打开 `malicious.html` 在一个新的标签页上(使用 `target=_blank`). 本身，这并不令人兴奋。  

但是，`malicious.html` 文档在新的标签页有一个 `window.opener` 属性指向当前你正在看的HTML文档的 `window` 对象，例如这个: `index.html`  

这意味着,如果用户点击了这个链接，`malicaous.html` 能够完全的控制当前文档的 `window` 对象。  

请记住即使 `index.html` 和 `malicaous.html` 属于不同的域这也是有效的，`window.opener.location` 跨域情况下依然能获取到！(虽然 `window.opener.document` 在跨域的情况下获取不到，[CORS](https://fetch.spec.whatwg.org/#http-cors-protocol)也是无效的)，以下是一个跨域的链接：  

```
<a class="user-generated" href="https://mathiasbynens.be/demo/opener" target="_blank">
<b>Click me!!1 (跨域)</b>
</a>
```

`malicious`将当前页面 `index.html` 换成了 `index.html#hax`, 这使得当前页面展示了一个隐藏信息。这是个相对来说无害的例子，但是取而代之的可以是重定向到一个钓鱼网站，设计得就像真的 `index.html` 一样，要求你登录。用户不会注意到这些，因为注意力在新开的 `malicaous` 页面，然而重定向发生在后台。攻击可以通过在重定向到钓鱼网站之前增加延迟使得更为巧妙([查看](http://www.azarask.in/blog/post/a-new-type-of-phishing-attack/)).  

如果 `window.opener` 被设置了，一个页面可以无视安全源触发 `opener` 页面的导航。  

## 2. 推荐做法

为了避免页面滥用 `window.opener` 使用 `rel=opener`,在 `Chrome 49 & Opear 36, Firefox 52, Desktop Safari 10.1+ 和 iOS Safari 10.3+`中 它能够保证 `window.opener` 是 `null`.  

```
<a class="user-generated" href="malicious.html" target="_blank" rel="noopener">
<b>Click me!!1 (使用 rel=noopener)</b>
</a>
```

对于更老的浏览器，你可以设置 `rel=noreferer`， 这样也能禁用掉 `Referer` 的 HTTP 头, 或者以下的 JavaScript 代码可能会触发弹出式窗口的拦截器：  

```
var otherWindow = window.open();
otherWindow.opener = null;
otherWindow.location = url;
```

```
<a class="user-generated" href="malicious.html" target="_blank" rel="noreferrer">
<b>Click me!!1 (now with <code>rel=noreferrer</code>-based workaround)</b>
</a>
```

```
<a class="user-generated" href="malicious.html" target="_blank" onclick="var otherWindow = window.open(); otherWindow.opener = null; otherWindow.location = href; return false;">
<b>Click me!!1 (now with <code>window.open()</code>-based workaround)</b>
</a>
```

记住 [基于JavaScript的方式在Safari上无效](https://github.com/danielstjules/blankshield#solutions),为了 `Safari` 的支持，插入一个隐藏的 `iframe` 去打开新的页面，然后立刻移除 `iframe`.  

不要使用 `target="_blank"`(或者其他的 `target` 打开新的导航上下文)，尤其是对于那些用户生成的链接内容，除非你有[好的理由](https://css-tricks.com/use-target_blank/)这样做。  

## 3. 说明

在 `Safari Technology Preview 68`, 锚点上的 `target="_blank"` 意味这 `rel="noopener"`, 如果明确的选择保留 `window.opener`，使用 `rel="opener"`, [查看这里](https://github.com/whatwg/html/issues/4078)  

## 4. Bug 追随清单

- [Gecko/Firefox bug #1222516](https://bugzilla.mozilla.org/show_bug.cgi?id=1222516)
- [WebKit/Safari bug #155166](https://bugs.webkit.org/show_bug.cgi?id=155166)
- [Microsoft Edge feature request](https://wpdev.uservoice.com/forums/257854-microsoft-edge-developer/suggestions/12942405-implement-rel-noopener)
- [Chromium/Chrome/Opera bug #168988](https://bugs.chromium.org/p/chromium/issues/detail?id=168988)