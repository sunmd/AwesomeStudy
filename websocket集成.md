最近笔者新参与的一个web项目，拟定采用vue2.0来编写，期间遇到有关使用websocket的问题，记录一下，个中遇到的一些问题和解决方法，分享给有需要的人。
 　　首先说一下vue2.0的初学体验，目前感觉上手还是很快的，对比其他框架比如angularjs，react等，的确是轻量级很多，并且确实如作者尤大大所诉，真实体会到了其[渐进式前端解决方案的思想](https://link.jianshu.com/?t=http://mp.weixin.qq.com/s/wcgOatZypjJTLek647Glpg) ，你完全可以根据项目的实际情况，选择性的采用最适合的模块和组件组合来更快速实现自己的功能，然后随着业务功能的扩展，渐进式的调整系统的架构，或采用更适合当前情况的架构。
 　　结合项目的实际情况，项目本身不大，初期版本定位是一个小型的单页面应用，目前团队前端开发人员都有一定的angularjs的开发基础，此时切换到vue上面来，也挺顺利的，既然是复杂度不是特别高的单页面应用，初期定位就是用vue-cli来构建项目的基本结构，然后配上vue-route路由，vue-resource请求API数据，很快页面雏形出来了。
 吧啦吧啦说了这么多，貌似还没切入主题，websocket，嗯，现在开始切入（咳咳，容我缓缓，刚刚还沉浸在尤大大的vue里面）。
 　　笔者不是专业的前端，对websocket还不是很了解，项目里面有一个应用场景是要在页面上实时显示服务端推送过来的数据，很显而易见的的服务端给客户端推送消息的场景，大家自然想到了websocket，而我但是对这个还是不是很清楚的，迅速查资料，[知乎上这篇文章](https://link.jianshu.com/?t=https://www.zhihu.com/question/20215561) 貌似是讲的不错，小白表示看的很过瘾。
 　　这个功能，服务端也采用了websocket，这需要客户端也相应的采用websocket来与客户端建立ws连接，这样客户端和服务端就可以双向的发送数据。笔者的定式思维是，一拿到这个就会觉得是不是又需要npm安装什么包似的，因为这个功能是从另一个项目（暂且称其为项目B）的功能迁移过来的，而项目B前端是用的angularjs开发的，前端直接用的angularjs的一个websocket的包（angular-websocke），然后几行代码就敲定了，创建连接，发送数据，接收数据，很是方便。然后切换到vue里面，很自然的就去搜有木有vue的websocket包，很快找到了vue-websocket，安装好后，调试，直接报错：

> ```
> XMLHttpRequest cannot load http://192.168.0.239:9000/socket.io/?EIO=3&transport=polling&t=LbjddEK. No 'Access-Control-Allow-Origin' header is present on the requested resource. Origin 'http://localhost:8080' is therefore not allowed access. The response had HTTP status code 404.
> ```

笔者对比了项目B，同样在客户端启动本地server，能正常创建到服务端的ws连接，不会像http请求一样有跨域的问题，同时另一问题就是，明明是要创建ws连接，为何浏览器的控制台里会提示是http的连接？
 vue-websocket没有调试成功，马上去找找有没有通用的包，于是在github找了js相关的websocket的相关包，比如ws，笔者对照了angularjs-websocket包，它也是依赖的ws。于是按照ws的说明文档，安装，然后webpack直接报错了：

> ```
> ERROR in ./~/ws/lib/WebSocketServer.js Module not found: Error: Cannot resolve module 'tls' in xx\node_modules\ws\lib @ ./~/ws/lib/WebSocketServer.js 15:10-24 ERROR in ./~/options/lib/options.js Module not found: Error: Cannot resolve module 'fs' in xx\node_modules\options\lib @ ./~/options/lib/options.js 6:9-22
> ```

查找资料，说是这两个包是服务端node用的，客户端没有所以导致报错，需要修改webpack的配置文件，使其不加载客户端不需要的包
[https://github.com/webpack/react-starter/issues/3#issuecomment-53395089](https://link.jianshu.com/?t=https://github.com/webpack/react-starter/issues/3#issuecomment-53395089)
 然后，webpack不报错了，但是前端调用ws生成实例的时候，依然会报错，因为ws链接创建的时候，先发起了http的链接请求，按照ws文档



```jsx
var WebSocket = require('ws')
var ws = new WebSocket('ws://xxx/ws', 'ws')
```

在在代码里，生成ws实例的时候，还是会产生http的链接，跟前面用vue-websocket的情况类似，因为是在本地启动的前端服务，所以这样发起到服务器的请求，会导致跨越报错。
 奇怪的是项目B里的angular使用的angular-websocket包，同样是基于ws包做的，在本地启动，是直接创建的ws链接，没有跨越的问题。
 　　然后在vue论坛上查找相关资料，类似的问题比如：
`[https://forum-archive.vuejs.org/topic/1293/webpack-bundling-issue-with-websockets/3](https://forum-archive.vuejs.org/topic/1293/webpack-bundling-issue-with-websockets/3)`
 有人提到：Can't you just use window.WebSocket instead of ws?
 提醒到了我，我摒弃了ws，直接用window.WebSocket，来创建ws连接，果然成功了。那为啥angular-websocket基于ws，能正常在浏览器执行，vue这边直接用ws不行，于是去查看angular-websocket的源码，才发现angular-websocket基于ws做了封装，它提供的$websocket对象，在当前浏览器环境下，本质上也是window.WebSocket的对象。



```jsx
import angular from 'angular';
var Socket;
if (typeof window === 'undefined') {
try {
var ws = require('ws');
Socket = (ws.Client||ws.client||ws);
} catch(e) {}
}
// Browser
Socket = (Socket || window.WebSocket || window.MozWebSocket);
////////////////////////////////////////////////////////////
function $WebSocketBackendProvider($log) {
this.create = function create(url, protocols) {
var match = /wss?:\/\//.exec(url);
if (!match) {
throw new Error('Invalid url provided');
}
if (protocols) {
return new Socket(url, protocols);
}
return new Socket(url);
};
```

看源码里面，angular-websocket也是用window.WebSocket创建的，只有在没有window对象的时候，才会基于ws包来创建websocket对象。然后瞬间感觉找准了方向，目前我们的场景主要是PC端的浏览器使用，可以直接用window.WebSocket对象来，于是查阅了window.WebSocket的文档
[https://developer.mozilla.org/en-US/docs/Web/API/WebSocket](https://link.jianshu.com/?t=https://developer.mozilla.org/en-US/docs/Web/API/WebSocket)，
 很快问题就解决了，原来自带的window对象就已经可以解决问题了，自己绕了一大圈，还是回到原点，细看了它文档里写的相关方法，确实和前面讲到的angular-websocket很像，确实angular-websocket是对这些方法，比如onopen，send，onmessange等方法进行了封装，让开发可以更方便的调用。
 　　另外，还有一个websocket相关的库[Socket.IO](https://link.jianshu.com/?t=http://socket.io/)，Socket.IO是一个跨平台，支持多种连接方式，如websocket，flashsocket,ajax等，如果客户端不支持websocket时，可以切换使用其他链接方式，比如ajax轮询等，Socket.IO完全由JavaScript实现、基于Node.js、支持WebSocket的协议用于实时通信、跨平台的开源框架，它包括了**客户端的JavaScript**和**服务器端的Node.js**，在客户端环境不支持websocket的情况下，用Socket.IO可以通过其他方式，模拟出websocket的效果。
 　　但是，笔者目前的情况是，服务端不是nodeJs环境，笔者在客户端安装了Socket.IO，并且按照文档说明，去建立ws链接，发现以下问题：

1. 安装socket.io-client，安装完之后，webpack报错
   `ERROR in ./~/socket.io-client/~/component-emitter/index.js Module build failed: Error: ENOENT: no such file or directory, open 'xx\node_modules\socket.io-client\node_modules\component-emitter\index.js' at Error (native) @ ./~/socket.io-client/lib/manager.js 8:14-42 ERROR in ./~/engine.io-client/~/component-emitter/index.js Module build failed: Error: ENOENT: no such file or directory, open 'xx\node_modules\engine.io-client\node_modules\component-emitter\index.js' at Error (native) @ ./~/engine.io-client/lib/socket.js 6:14-42`
    参考[https://github.com/socketio/socket.io-client/issues/933](https://link.jianshu.com/?t=https://github.com/socketio/socket.io-client/issues/933)这个需要修改一下webpack的配置文件，添加以下两行：



```css
module: {
    noParse: ['ws']
  },
  externals: ['ws']
```

这样webpack可以打包通过

1. 然后测试，创建连接`var socket = io(url);`，这样创建的链接，Socket.IO默认是按轮询方式发起的http请求（很奇怪，当前浏览器明明是支持websocket的），这样首先就出现了前面的http跨域请求报错。
   `XMLHttpRequest cannot load http://192.168.0.239:9000/socket.io/?EIO=3&transport=polling&t=LbjddEK. No 'Access-Control-Allow-Origin' header is present on the requested resource. Origin 'http://localhost:8080' is therefore not allowed access. The response had HTTP status code 404.`
    查阅资料，创建连接的时候，可以指定参数`io(WS_URL, {transports: ['websocket', 'polling', 'flashsocket']})`，设置其发起websocket链接，这样在console里看到的确实是发起的ws请求，然而依然报404错：
   `WebSocket connection to 'ws://192.168.0.239:9000/socket.io/?EIO=3&transport=websocket' failed: Error during WebSocket handshake: Unexpected response code: 404`
    提示是404找不到，确实是服务端没有`'ws://192.168.0.239:9000/socket.io/?EIO=3&transport=websocket'`这样的url，这种url是默认的ws服务端的url，可以设置 path，path默认是`/socket.io`，设置path后，`io(WS_URL, {path: '/ws', transports: ['websocket', 'polling', 'flashsocket']})`，依然报301错：
   `WebSocket connection to 'ws://192.168.0.239:9000/ws/?EIO=3&transport=websocket' failed: Error during WebSocket handshake: Unexpected response code: 301`
    笔者对比了成功建立ws链接时的url，`ws://192.168.0.239:9000/ws`，然后用工具测试了下，只有path是`/ws`，即[ws://192.168.0.239:9000/ws?EIO=3&transport=websocket](https://link.jianshu.com/?t=ws://192.168.0.239:9000/ws?EIO=3&transport=websocket)`这样的url才能成功建立ws链接，这里报301的错误，是因为socketio会在path后面添加`/?EIO=3&transport=websocket`这样一串参数，这样导致了服务端路由匹配不上，导致出现了404或301的情况，所以这里需要服务端配合调整一下路由规则，即可。

最终，结合当前的实际情况还是采用了原生的websocket，Socket.IO对url的格式有一定的规范要求，比较适合客户端和服务端都同时可控的情况，目前查了socketio的api文档，也没找到合适的方法，可以自定义url，大家有什么好的解决方法，请不吝赐教！
 　　今天码字就码到这里，就酱紫了，欢迎大家来拍砖！



作者：七宝琥珀
链接：https://www.jianshu.com/p/0d20a032d0ec
来源：简书
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。



# [我可以使用socket.io-client连接到标准websocket吗？(can I use socket.io-client to connect to a standard websocket?)](https://www.it1352.com/529777.html)

这意味着它从与Socket.io客户端/服务器交互的客户端/服务器代码的角度看起来类似于WebSocket.然而,网络流量看起来与一个简单的WebSocket非常不同 – 除了连接WebSocket之外,建立一个更强大的协议之外,还有一个初始的握手方式.握手描述为[here](javascript:void())和消息协议[here](javascript:void())(均为Socket.IO协议规范的链接).

如果你正在编写一个WebSocket服务器,你最好只是使用裸机WebSocket接口而不是Socket.io客户端,除非你打算实现所有的Socket.io协议.