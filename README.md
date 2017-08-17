# javascript-sdk-design-cn
[javascript-sdk-design](https://github.com/hueitan/javascript-sdk-design) 的中文版

## 介绍
本文介绍如何对开发 JavaScript 网页应用设计SDK，适用于桌面端，移动端，不同平台，不同浏览器。对于JavaScript实现的非网页应用（硬件，嵌入式，node/io js）场景则不适用，这些场景会在未来介绍。

由于我没有找到比较好的 JavaScript SDK 文档，所以把我的个人经验做了整理和记录。JavaScript-SDK-Design 不仅仅介绍SDK，还包括用户和浏览器的关系。写的越多，想的越多，就开始关注不同平台，不同浏览器之间的性能和差异。

## 什么是 SDK

SDK是软件开发工具包 的缩写，是能够让编程者开发出应用程序的软件包。一般SDK包括一个或多个API、开发工具集和说明文档。

## 设计哲学

SDK设计成什么样子取决于SDK用途，但是必须要原生 ，简短 ，执行迅速 ，代码干净 ，易读 ，可测试 。

用原生 Javascript 写代码，不推荐使用Livescript, Coffeescript, Typescript 这种编译生成 JS 的语言。用原生 Javascript 写是一个更快更好的方式。

尽量不要使用jQuery，而应该使用轻量的类库代替。如果是DOM操作可以使用zepto.js 。如果要发HTTP ajax请求 使用window.fetch 。

SDK版本一旦发布，要保证可以兼容旧版本并且要能被将来的版本兼容。所以要给SDK写文档 、写注释、做单元测试和情景测试。

## 范围

参考书籍:[第三方JavaScript](http://thirdpartyjs.com/)

哪些情况你应该设计SDK？

- 1. 嵌入的widgets - 在第三方网页上嵌入的交互应用(Disqus, Google Maps, Facebook Widget)

- 2. 分析和度量 - 收集用户信息，了解访客和网站交互的方式 (GA, Flurry, Mixpanel)。

- 3. 封装网络服务 - 开发调用外部网站服务的客户端应用. (Facebook Graph API)

还有哪些情况应该使用SDK ？可以提建议 。

应该使用异步语法来加载脚本。 应该改善用户体验，SDK类库不应该影响主页面的加载。

#### 异步语法

```
<script>
  (function () {
    var s = document.createElement('script');
    s.type = 'text/javascript';
    s.async = true;
    s.src = 'http://xxx.com/sdk.js';
    var x = document.getElementsByTagName('script')[0];
    x.parentNode.insertBefore(s, x);
  })();
</script>
```
针对现代浏览器，可以使用async。

```
<script async src="http://xxx.com/sdk.js"></script>
```
#### 传统语法
```
<script type="text/javascript" src="http://xxx.com/sdk.js"></script>
```

#### 比较
下列图标表示异步语法和同步语法的差别。

异步:

```
 |----A-----|
    |-----B-----------|
        |-------C------|
```

同步:

```
|----A-----||-----B-----------||-------C------|
```

#### 异步的问题

如果异步加载，不能像下列代码一样调用SDK。
```
<script>
  (function () {
    var s = document.createElement('script');
    s.type = 'text/javascript';
    s.async = true;
    s.src = 'http://xxx.com/sdk.js';
    var x = document.getElementsByTagName('script')[0];
    x.parentNode.insertBefore(s, x);
  })();

  // execute your script immediately here
  SDKName('some arguments');
</script>
```
这样做会导致未知的结果，因为SDKName()执行的时候尚未被加载完成。
```
<script>
  (function () {
    // add a queue event here
    SDKName = SDKName || function () {
      (SDKName.q = SDKName.q || []).push(arguments);
    };
    var s = document.createElement('script');
    s.type = 'text/javascript';
    s.async = true;
    s.src = 'http://xxx.com/sdk.js';
    var x = document.getElementsByTagName('script')[0];
    x.parentNode.insertBefore(s, x);
  })();

  // execute your script immediately here
  SDKName('some arguments');
</script>
```
或者 使用[].push
```
<script>
  (function () {
    // add a queue event here
    SDKName = window.SDKName || (window.SDKName = []);
    var s = document.createElement('script');
    s.type = 'text/javascript';
    s.async = true;
    s.src = 'http://xxx.com/sdk.js';
    var x = document.getElementsByTagName('script')[0];
    x.parentNode.insertBefore(s, x);
  })();

  // execute your script immediately here
  SDKName.push(['some arguments']);
</script>
```

#### 其他

还有其他方式加载代码。

**ES2015中的Import**
```
import "your-sdk";
```
**使用Modular**

完整代码请看Loading JavaScript Modules。
```
module('sdk.js',['sdk-track.js', 'sdk-beacon.js'],function(track, beacon) {
  // sdk definitions, split into local and global/exported definitions
  // local definitions
  // exports
});

// you should contain this "module" method
(function () {

  var modules = {}; // private record of module data

  // modules are functions with additional information
  function module(name,imports,mod) {

    // record module information
    window.console.log('found module '+name);
    modules[name] = {name:name, imports: imports, mod: mod};

    // trigger loading of import dependencies
    for (var imp in imports) loadModule(imports[imp]);

    // check whether this was the last module to be loaded
    // in a given dependency group
    loadedModule(name);
  }

  // function loadModule
  // function loadedModule

  window.module = module;
})();
```

## SDK版本管理

不要用类似 brand-v<timestamp>.js brand-v<datetime>.js brand-v1-v2.js的版本号，这样会导致SDK使用者不知道最新的版本是什么。

使用 [Semantic Versioning ](http://semver.org/) “主版本.小版本.补丁号”这种有语义的命名方式管理版本。v1.0.0 v1.5.0 v2.0.0这样的版本号让使用者容易在changelog文档中跟综和查找。

通常情况下，我们可以通过不同的方式来说明SDK版本，这取决于您的服务和设计

**Using Query String path.**

http://xxx.com/sdk.js?v=1.0.0

**Using the Folder Naming.**

http://xxx.com/v1.0.0/sdk.js

**Using hostname (subdomain).**
http://v1.<DOMAIN>.com/sdk.js
  
## Changelog Document 日志文档

如果SDK有升级，应该通知SDK使用者。major，minor版本甚至是修改bug都应该写Changelog Document。 使用者会有一个好的体验。 - Keep a Changelog, Github Repo

每个版本都应该有：
```
[Added] for new features.
[Changed] for changes in existing functionality.
[Deprecated] for once-stable features removed in upcoming releases.
[Removed] for deprecated features removed in this release.
[Fixed] for any bug fixes.
[Security] to invite users to upgrade in case of vulnerabilities.
```

## 命名空间

应该最多定义一个命名空间，不要使用通用的名字定义命名空间以防止和其他类库冲突。

应该用(function () { ... })()把SDK代码包起来。

jQuery, Node.js等等类库经常使用的一个方法是把创造私有命名空间的整个文件用闭包包起来，这样可以避免和其他模块冲突。

避免 命名空间冲突

参考Google Analytics项目你可以通过改变ga的值来定义你的命名空间。
```
(function(i,s,o,g,r,a,m){i['GoogleAnalyticsObject']=r;i[r]=i[r]||function(){
(i[r].q=i[r].q||[]).push(arguments)},i[r].l=1*new Date();a=s.createElement(o),
m=s.getElementsByTagName(o)[0];a.async=1;a.src=g;m.parentNode.insertBefore(a,m)
})(window,document,'script','//www.google-analytics.com/analytics.js','ga');
```
OpenX 的经验是提供一个参数来请求相关命名空间。

<script src="http://your_domain/sdk?namespace=yourcompany"></script>


## 存储机制

#### Cookies
考虑subdomain 和 path 情况，使用Cookie的范围非常复杂。

对于path=/， 在 http://github.com ，cookie有first=value1 在http://sub.github.com， 有另一个cookie:second=value2

|               | http://github.com | http://sub.github.com |
|:-------------:|:-----------------:|:---------------------:|
| first=value1  |         ✓         |           ✓           |
| second=value2 |         ✘         |           ✓           |


first=value1 在 http://github.com 生效, second=value2 在 http://github.com/path1 生效， third=value3 在 http://sub.github.com 生效,

|               | http://github.com | http://github.com/path1 | http://sub.github.com |
|:-------------:|:-----------------:|:-----------------------:|:---------------------:|
| first=value1  |         ✓         |            ✓            |           ✓           |
| second=value2 |         ✘         |            ✓            |           ✘           |
| third=value3  |         ✘         |            ✘            |           ✓           |


**检查Cookie可写性**

给定一个域（假设是当前主机名），检查cookie是否可写。
```
var checkCookieWritable = function(domain) {
    try {
        // Create cookie
        document.cookie = 'cookietest=1' + (domain ? '; domain=' + domain : '');
        var ret = document.cookie.indexOf('cookietest=') != -1;
        // Delete cookie
        document.cookie = 'cookietest=1; expires=Thu, 01-Jan-1970 00:00:01 GMT' + (domain ? '; domain=' + domain : '');
        return ret;
    } catch (e) {
        return false;
    }
};
```
#### Session
客户端JavaScript代码无法写session，请参考服务端实现。

浏览器打开页面，session一直有效，页面的重新加载和恢复，session也不会被删除。在新tab页或者窗口中打开页面会导致新的session初始化。

#### 本地存储
存储的数据没有有效期，数据的额度可以很多（至少5M）并且不会转到服务端。

相同域的本地存储不能共享，可以在站点内部创建框架并且可以用postMessage在本地存储之间传递数据。怎么做?

检查本地存储可写行

不是每个浏览器都支持window.localStorage，SDK在使用之前必须确认是否可用。

```
var testCanLocalStorage = function() {
   var mod = 'modernizr';
   try {
       localStorage.setItem(mod, mod);
       localStorage.removeItem(mod);
       return true;
   } catch (e) {
       return false;
   }
};
```

#### Session 存储

为session存储数据（当tab页关闭，数据失效）。

#### LocalStorage

**检查SessionStorage可写性**
```
var checkCanSessionStorage = function() {
  var mod = 'modernizr';
  try {
    sessionStorage.setItem(mod, mod);
    sessionStorage.removeItem(mod);
    return true;
  } catch (e) {
    return false;
  }
}
```

## 事件
浏览器端有load unload on off bind 事件，这里有一些代码可以处理不同浏览器的差异。

#### Document Ready
在开始执行SDK功能之前要先确保整个页面加载完成。

```
// handle IE8+
function ready (fn) {
    if (document.readyState != 'loading') {
        fn();
    } else if (window.addEventListener) {
        // window.addEventListener('load', fn);
        window.addEventListener('DOMContentLoaded', fn);
    } else {
        window.attachEvent('onreadystatechange', function() {
            if (document.readyState != 'loading')
                fn();
            });
    }
}
```
在document已经被完全加载和解析后执行，不用等stylesheets，images和子模块完成加载。

load事件可以用来探测页面是否完全加载

#### 消息事件
关于iframe和窗口的跨源通信，请读API文档。

```
// in the iframe
parent.postMessage("Hello"); // string

// ==========================================

// in the iframe's parent
// Create IE + others compatible event handler
var eventMethod = window.addEventListener ? "addEventListener" : "attachEvent";
var eventer = window[eventMethod];
var messageEvent = eventMethod == "attachEvent" ? "onmessage" : "message";

// Listen to message from child window
eventer(messageEvent,function(e) {
  // e.origin , check the message origin
  console.log('parent received message!:  ',e.data);
},false);
```

发送的消息格式应该是String，如果用json做一些高级用法，就用JSON String。虽然很多浏览器支持对参数的结构化克隆算法 ，但并不是全部浏览器都支持。

#### 方向改变
探测设备方向改变。

window.addEventListener('orientationchange', fn);
得到方向旋转角度。

window.orientation; // => 90, -90, 0
竖屏正方向，竖屏反方向，横屏正方向，横屏反方向。（实验性的）

// https://developer.mozilla.org/en-US/docs/Web/API/Screen/orientation
var orientation = screen.orientation || screen.mozOrientation || screen.msOrientation;
#### 禁止滚动
电脑页面用CSS代码overflow: hidden，移动页面不支CSS这种写法，用javascript事件。


```js
document.addEventListener('touchstart', function(e){ e.preventDefault(); }); // prevent scroll
// or
document.body.addEventListener('touchstart', function(e){ e.preventDefault(); }); // prevent scroll
// use move if you need some touch event
document.addEventListener('touchmove', function(e){ e.preventDefault(); }); // prevent scroll
```

## 请求
我们的SDK用Ajax请求和服务器通信，虽然可以用jQuery ajax请求，但这里我们有更好的方案实现它。

#### Image Beacon
使用Image Beacon让浏览器通过GET 请求获取图片。

记住加上时间戳防止浏览器缓存

(new Image()).src = 'http://xxxxx.com/collect?id=1111';
需要注意，GET Query字符串有长度限制，一般是2048，取决于浏览器和服务器。你应该检查请求是否超长了。

if (length > 2048) {
    // do Multiple Post (form)
} else {
    // do Image Beacon
}
你可能在encodeURI 和encodeURIComponent上遇到问题，最好能理解它们。见下

图片加载成功/失败回调函数。

var img = new Image();
img.src = 'http://xxxxx.com/collect?id=1111';
img.onload = successCallback;
img.onerror = errorCallback;
#### 单一Post
用表单元素的POST方法发送键值对。
```
var form = document.createElement('form');
var input = document.createElement('input');

form.style.display = 'none';
form.setAttribute('method', 'POST');
form.setAttribute('action', 'http://xxxx.com/track');

input.name = 'username';
input.value = 'attacker';

form.appendChild(input);
document.getElementsByTagName('body')[0].appendChild(form);

form.submit();
```
#### 多重Post
服务一般很复杂，我们需要发送POST发送很多数据。
```
function requestWithoutAjax( url, params, method ){

    params = params || {};
    method = method || "post";

    // function to remove the iframe
    var removeIframe = function( iframe ){
        iframe.parentElement.removeChild(iframe);
    };

    // make a iframe...
    var iframe = document.createElement('iframe');
    iframe.style.display = 'none';

    iframe.onload = function(){
        var iframeDoc = this.contentWindow.document;

        // Make a invisible form
        var form = iframeDoc.createElement('form');
        form.method = method;
        form.action = url;
        iframeDoc.body.appendChild(form);

        // pass the parameters
        for( var name in params ){
            var input = iframeDoc.createElement('input');
            input.type = 'hidden';
            input.name = name;
            input.value = params[name];
            form.appendChild(input);
        }

        form.submit();
        // remove the iframe
        setTimeout( function(){
            removeIframe(iframe);
        }, 500);
    };

    document.body.appendChild(iframe);
}
requestWithoutAjax('url/to', { id: 2, price: 2.5, lastname: 'Gamez'});
```
#### Iframe
如果需要在页面中生成内容，你可以用iframe把你的html嵌入进来。
```
var iframe = document.createElement('iframe');
var body = document.getElementsByTagName('body')[0];

iframe.style.display = 'none';
iframe.src = 'http://xxxx.com/page';
iframe.onreadystatechange = function () {
    if (iframe.readyState !== 'complete') {
        return;
    }
};
iframe.onload = loadCallback;

body.appendChild(iframe);
```
把iframe内部额外的margin删除

```
<iframe src="..."
 marginwidth="0"
 marginheight="0"
 hspace="0"
 vspace="0"
 frameborder="0"
 scrolling="no"></iframe>
 ```
把html内容放到iframe
```
<iframe id="iframe"></iframe>

<script>
  var html_string= "content <script>alert(location.href);</script>";
  document.getElementById('iframe').src = "data:text/html;charset=utf-8," + escape(html_string);
  // alert data:text/html;charset=utf-8.....
  // access cookie get ERROR

  var doc = document.getElementById('iframe').contentWindow.document;
  doc.open();
  doc.write('<body>Test<script>alert(location.href);</script></body>');
  doc.close();
  // alert "top window url"

  var iframe = document.createElement('iframe');
  iframe.src = 'javascript:;\'' + encodeURI('<html><body><script>alert(location.href);</body></html>') + '\'';
  // iframe.src = 'javascript:;"' + encodeURI((html_tag).replace(/\"/g, '\\\"')) + '"';
  document.body.appendChild(iframe);
  // alert "about:blank"
</script>
```
#### jsonp脚本
是指你的服务器需要返回JavaScript代码以让浏览器执行，只包括JS脚本链接。

  (function () {
    var s = document.createElement('script');
    s.type = 'text/javascript';
    s.async = true;
    s.src = '/yourscript?some=parameter&callback=jsonpCallback';
    var x = document.getElementsByTagName('script')[0];
    x.parentNode.insertBefore(s, x);
  })();

更多关于jsonp信息

- 1. JSONP只能在GET请求工作。

- 2. JSONP缺少错误处理，意味着你不能在返回码404,500等里面探测。

- 3. JSONP请求通常是异步的。

- 4. 注意CSRF攻击。

- 5. 跨域通讯，脚本回应端（服务端）不需要关心CORS。

#### Navigator.sendBeacon()
先看下文档

This method addresses the needs of analytics and diagnostics code that typically attempt to send data to a web server prior to the unloading of the document. Sending the data any sooner may result in a missed opportunity to gather data. However, ensuring that the data has been sent during the unloading of a document is something that has traditionally been difficult for developers.

通过api发POST beacon请求，很酷。

navigator.sendBeacon("/log", analyticsData);
XMLHttpRequest
写XMLHttpRequest 不是好主意。我假设你不想在和浏览器斗争上浪费时间，你可以试下这些代码。

1. window.fetch - A window.fetch JavaScript polyfill. 2. got - Simplified HTTP/HTTPS requests 3. microjs - list of ajax lib 4. more

####  Fragment Identifier
#这是哈希标志。记住有哈希标志的请求，哈希标志最终不会发出去。

比如，你在页面http://github.com/awesome#huei90中。

// Sending a request with a parameter url which contains current url
(new Image()).src = 'http://yourrequest.com?url=http://github.com/awesome#huei90';

// actual request will be without #
(new Image()).src = 'http://yourrequest.com?url=http://github.com/awesome';

// Solved, encodeURIComponent(url);
(new Image()).src = 'http://yourrequest.com?url=' + encodeURIComponent('http://github.com/awesome#huei90');

#### 最大连接数
检查浏览器的最大连接请求数浏览器范围。

<h2 align="center">
 <img src="https://cloud.githubusercontent.com/assets/2560096/9082891/ac4dc26e-3b99-11e5-8178-606270a801c4.png" alt="max number of connection"/>
</h2>

## Component of URI


很重要的一点，你应该知道SDK是否需要解析本地url。
```
                         authority
                   __________|_________
                  /                    \
              userinfo                host                          resource
               __|___                ___|___                 __________|___________
              /      \              /       \               /                      \
         username  password     hostname    port     path & segment      query   fragment
           __|___   __|__    ______|______   |   __________|_________   ____|____   |
          /      \ /     \  /             \ / \ /                    \ /         \ / \
    foo://username:password@www.example.com:123/hello/world/there.html?name=ferret#foo
    \_/                     \ / \       \ /    \__________/ \     \__/
     |                       |   \       |           |       \      |
  scheme               subdomain  \     tld      directory    \   suffix
                                   \____/                      \___/
                                      |                          |
                                    domain                   filename
```

#### 解析URI
(https://developer.mozilla.org/en-US/docs/Web/API/Window/URL). 这有个很简单的方法，是通过本地URL()接口，但是并不支持所有浏览器尚未标准化。

```
var parser = new URL('http://github.com/huei90');
parser.hostname; // => "github.com"
```
那些不支持URL()接口的浏览器，尝试DOM createElement('a')方法。

```
var parser = document.createElement('a');
parser.href = "http://github.com/huei90";
parser.hostname; // => "github.com"
```
## 调试

#### 模拟多重网域
你不需要注册不同域名来模拟多网域，只需要编辑操作系统的hosts文件。
```
$ sudo vim /etc/hosts
```
加上下面条目。
```
# refer to localhost
127.0.0.1 publisher.net
127.0.0.1 sdk.net
```
这时你可以访问http://publisher.net 和http://sdk.net页面了。

#### 开发工具
如果需要调试SDK，使用浏览器提供的调试工具- Chrome Developer Tools Safari Developer Tools Firebug，开发工具简称DevTools。

开发者可以通过DevTools访问应用和浏览器内部。使用DevTools高效跟踪布局问题，设置JavaScript断点，得到代码优化的洞察力。

#### Console Logs
测试预期文本输出，或者其他调试，可以用浏览器接口console.log()做Console Logs。有许多方法格式化和输出你的信息，查找更多[Console接口](https://developer.mozilla.org/en/docs/Web/API/console)。

![screen shot 2015-06-15 at 3 50 23 pm](https://cloud.githubusercontent.com/assets/2560096/8155377/411fb24a-1376-11e5-98da-f71f8ed29bcd.png)

#### Debugging Proxy
有时候调试代理帮你测试你的SDK。调试网络拥塞，修改cookies，报头，缓存，编辑http request/response，SSL代理，ajax调试等等。

这里有些软件你可以试一下。
- [FiddlerCore](http://www.telerik.com/fiddler/fiddlercore)
- [Charles](http://www.charlesproxy.com)
- [Cellist](https://itunes.apple.com/tw/app/cellist-http-debugging-proxy/id897814548?l=zh&mt=12)
#### BrowserSync
[BrowserSync](http://www.browsersync.io/)通过同步多设备文件的变化和相互作用的方法让你的测试更快。非常快并且是免费的。

如果需要测试SDK在各种设备上的结果，它可以帮你很多。试一下=)


## 小贴士和诀窍
#### Piggyback
有时候不希望开发者包含所有SDK源，只需要做一个1x1像素的请求。我们只需要要求开发者包含我们链接的图片文件。
```
<img height="1" width="1" alt="" style="display:n/>
```
#### 页面可见性接口
                                        
有时SDK像知道页面是否取得用户焦点。试试代码 `visibly.js` 和 `visibilityjs`。

#### Document Referrer
用document.referrer得到上一个页面的url。注意上一个是指浏览器上一个而不是人们认为的上一个。举个例子，加入你点击浏览器后退按钮，页面A -> 页面B -> 页面C -> (后退) 页面B，现在页面B的上一个是页面A，不是页面C。

#### Console Logs Polyfill
这不是真的代码，只是保证调用console.log接口不会在客户端抛出异常事件。
```
if (typeof console === "undefined") {
    var f = function() {};
    console = {
        log: f,
        debug: f,
        error: f,
        info: f
    };
}
```
#### EncodeURI or EncodeURIComponent
理解escape() encodeURI() encodeURIComponent()的区别，查看[这里](http://stackoverflow.com/a/3608791/1748884)。

记住encodeURI() 和 encodeURIComponent()之间有11个字符不同。 这些字符是# $ & + , / : ; = ? 


<h2 align="center">
 <img src="http://i.imgur.com/rHWC1r1.png" alt="encodeuri or encodeuricomponent"/>
</h2>
#### 用回调函数加载脚本

异步加载代码增加回调事件。
```
function loadScript(url, callback) {
  var script = document.createElement('script');
  script.async = true;
  script.src = url;

  var entry = document.getElementsByTagName('script')[0];
  entry.parentNode.insertBefore(script, entry);

  script.onload = script.onreadystatechange = function () {
    var rdyState = script.readyState;

    if (!rdyState || /complete|loaded/.test(script.readyState)) {
      callback();

      // detach the event handler to avoid memory leaks in IE (http://mng.bz/W8fx)
      script.onload = null;
      script.onreadystatechange = null;
    }
  };
}
```
#### 只能调用一次的函数
这里展示如何实现只能一次调用。

>偶尔你希望一个函数只能被调用一次。经常这些函数在事件监听列表，很难管理。当然你可以简单的把它从监听列表删除，但是有时候希望完美，你只是希望函数只能被调用一次。下面的JavaScript函数让它变为可能！

```
// Copy from DWB
// http://davidwalsh.name/javascript-once
function once(fn, context) { 
    var result;

    return function() { 
        if(fn) {
            result = fn.apply(context || this, arguments);
            fn = null;
        }

        return result;
    };
}

// Usage
var canOnlyFireOnce = once(function() {
    console.log('Fired!');
});

canOnlyFireOnce(); // "Fired!"
canOnlyFireOnce(); // nada
```
### Get Style Value

**Get inline-style value**

```html
<span id="black" style="color: black"> This is black color span </span>
<script>
    document.getElementById('black').style.color; // => black
</script>
```

**Get Real style value**

```html
<style>
#black {
    color: red !important;
}
</style>

<span id="black" style="color: black"> This is black color span </span>

<script>
    document.getElementById('black').style.color; // => black

    // real
    var black = document.getElementById('black');
    window.getComputedStyle(black, null).getPropertyValue('color'); // => rgb(255, 0, 0)
</script>
```

ref: [https://developer.mozilla.org/en-US/docs/Web/API/Window/getComputedStyle](https://developer.mozilla.org/en-US/docs/Web/API/Window/getComputedStyle)

#### 检查元素是否在观察窗
更多资料在 [这里](http://stackoverflow.com/questions/123999/how-to-tell-if-a-dom-element-is-visible-in-the-current-viewport/7557433#7557433).
```
function isElementInViewport (el) {

    //special bonus for those using jQuery
    if (typeof jQuery === "function" && el instanceof jQuery) {
        el = el[0];
    }

    var rect = el.getBoundingClientRect();

    return (
        rect.top >= 0 &&
        rect.left >= 0 &&
        rect.bottom <= (window.innerHeight || document.documentElement.clientHeight) && /*or $(window).height() */
        rect.right <= (window.innerWidth || document.documentElement.clientWidth) /*or $(window).width() */
    );
}
```
#### 检查元素可视性
```
var isVisible: function(b) {
    var a = window.getComputedStyle(b);
    return 0 === a.getPropertyValue("opacity") || "none" === a.getPropertyValue("display") || "hidden" === a.getPropertyValue("visibility") || 0 === parseInt(b.style.opacity, 10) || "none" === b.style.display || "hidden" === b.style.visibility ? false : true;
}

var element = document.getElementById('box');
isVisible(element); // => false or true
```
#### 得到视窗大小
```
var getViewportSize = function() {
    try {
        var doc = top.document.documentElement
          , g = (e = top.document.body) && top.document.clientWidth && top.document.clientHeight;
    } catch (e) {
        var doc = document.documentElement
          , g = (e = document.body) && document.clientWidth && document.clientHeight;
    }
    var vp = [];
    doc && doc.clientWidth && doc.clientHeight && ("CSS1Compat" === document.compatMode || !g) ? vp = [doc.clientWidth, doc.clientHeight] : g && (vp = [doc.clientWidth, doc.clientHeight]);
    return vp;
}

// return as rray [viewport_width, viewport_height]
```
