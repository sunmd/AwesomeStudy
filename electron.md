## 每一代的electron都会有一些废弃的接口要消息

## Planned Breaking API Changes (14.0)[](https://www.electronjs.org/docs/latest/breaking-changes#planned-breaking-api-changes-140)

### Removed: `remote` module[](https://www.electronjs.org/docs/latest/breaking-changes#removed-remote-module)

The `remote` module was deprecated in Electron 12, and will be removed in Electron 14. It is replaced by the [`@electron/remote`](https://github.com/electron/remote) module.

```js
// Deprecated in Electron 12:
const { BrowserWindow } = require('electron').remote
```

Copy

```js
// Replace with:
const { BrowserWindow } = require('@electron/remote')

// In the main process:
require('@electron/remote/main').initialize()
```

[API 参考 - 重要的API变更 - 《Electron 12.0 官方文档》 - 书栈网 · BookStack](https://www.bookstack.cn/read/electronjs-12.0-zh/743d4d999cdca50b.md)

### [Vue](https://so.csdn.net/so/search?from=pc_blog_highlight&q=Vue)2.6.11+electron13.0.0在渲染进程中使用remote，报错："TypeError: fs.existsSync is not a function"。

前言
Vue+electron项目，当在渲染进程中调用remote时导致页面空白，控制台报错TypeError: fs.existsSync is not a function

错误原因
渲染进程没有集成node环境因此无法使用fs包，以及无法使用require关键字。

解决
参考官方文档
1：在主进程中开启远程对象

​      
​      let win = new BrowserWindow({
​        show:false,
​        webPreferences: {
​    // Use pluginOptions.nodeIntegration, leave this alone
​      // See nklayman.github.io/vue-cli-plugin-electron-builder/guide/security.html#node-integration for more info
​      nodeIntegration: true,
​      contextIsolation: false,
​      //使用远程模块必须开启此处
​      enableRemoteModule:true
​    }
​     })

2：在渲染进程中引入remote

const {remote} = window.require('electron')
1
注：使用以下这种方式，无法获取remote。

const {remote} = require('electron')
1
问题解决。
————————————————
版权声明：本文为CSDN博主「模范生-」的原创文章，遵循CC 4.0 BY-SA版权协议，转载请附上原文出处链接及本声明。
原文链接：https://blog.csdn.net/weixin_38742935/article/details/119897371ps://blog.csdn.net/weixin_38742935/article/details/119897371

As the error suggests `contextIsolation` needs to be enabled to allow you to use the `contextBridge` API

Note: `contextIsolation` is enabled by default starting from Electron 12.0

## Node Integration

As of v2.0 of VCPEB, Electron `nodeIntegration` is disabled by default. This blocks all node APIs such as `require`. This reduces [security risks (opens new window)](https://electronjs.org/docs/tutorial/security#2-do-not-enable-nodejs-integration-for-remote-content), and is a recommended best practice by the Electron team. If you need to use native modules such as `fs` or `sqlite` in the renderer process, you can enable `nodeIntegration` in `vue.config.js`:

```js
module.exports = {
  pluginOptions: {
    electronBuilder: {
      nodeIntegration: true
    }
  }
}
```

IPC Without Node Integration

You can still use [IPC (opens new window)](https://www.electronjs.org/docs/api/ipc-renderer)without `nodeIntegration`. Just create a [preload script](https://nklayman.github.io/vue-cli-plugin-electron-builder/guide/guide.html#preload-files) with the following code:

```js
import { ipcRenderer } from 'electron'
window.ipcRenderer = ipcRenderer
```

Now, you can access `ipcRenderer` with `window.ipcRenderer` in your Vue app.

If you want to use IPC without `nodeIntegration` and with `contextIsolation`, use this:

```js
import { contextBridge, ipcRenderer } from 'electron'

// Expose protected methods that allow the renderer process to use
// the ipcRenderer without exposing the entire object
contextBridge.exposeInMainWorld('ipcRenderer', {
  send: (channel, data) => {
    // whitelist channels
    let validChannels = ['toMain']
    if (validChannels.includes(channel)) {
      ipcRenderer.send(channel, data)
    }
  },
  receive: (channel, func) => {
    let validChannels = ['fromMain']
    if (validChannels.includes(channel)) {
      // Deliberately strip event as it includes `sender`
      ipcRenderer.on(channel, (event, ...args) => func(...args))
    }
  }
})
```

Then, you can use `window.ipcRenderer.(send/receive)` in the renderer process.

(Solution from [this StackOverflow answer (opens new window)](https://stackoverflow.com/questions/55164360/with-contextisolation-true-is-it-possible-to-use-ipcrenderer/59675116#59675116))

preload需要添加两个地方

# [使用vue-cli-plugin-electron-builder打包后无法加载preload.js文件](https://www.cnblogs.com/cxddxg/p/15179143.html)

在使用`vue-cli-plugin-electron-builder`打包后发现应用程序没有执行preload.js文件于是查找原因

根据查阅的资料打包后的源文件放在`\dist_electron\win-unpacked\resources\app.asar`,`app.asar`是一个压缩文件需要解压才能看到里面的内容

```js
npm install -g asar
/**
    cd到app.asar所在的目录,将文件解压到file文件夹
**/
asar extract ./app.asar ./file
```

![img](https://img2020.cnblogs.com/blog/2054707/202108/2054707-20210824102504469-1962030845.png)

没有发现`preload.js`

**解决方案**

在`vue.config.js`中如下配置

```js
module.exports = {
  pluginOptions:{
    electronBuilder:{
      preload:'src/preload.js'
    }
  }
}
```

截取`background.js`的部分代码

```js
win = new BrowserWindow({
  webPreferences: {
    nodeIntegration: true,
    contextIsolation: false,
    webviewTag: true,
    preload: path.join(__dirname, '/preload.js')
  },
  resizable: false,
  frame: false
});
```

打包后解压发现`preload.js`出现，搞定~运行也没问题

![img](https://img2020.cnblogs.com/blog/2054707/202108/2054707-20210824102525444-545164888.png)

## vue.config.js

1. 如果使用vue+electron来进行打包

2. 配置会放置在vue.config.js中

3. ```js
   module.exports = {
     pluginOptions: {
       electronBuilder: {
         preload: "src/preload.js",
         builderOptions: {
           asar: false, // asar打包
           extraResources: {
             from: "./config.json",
             to: "../",
           },
         },
       },
     },
   };
   ```

4. `electronBuilder`这个字段会替换`build`字段

[Configuration | Vue CLI Plugin Electron Builder (nklayman.github.io)](https://nklayman.github.io/vue-cli-plugin-electron-builder/guide/configuration.html#typescript-options)

**When we work in electron vue js. we add new file vue.config.js and paste these line of code.**

```
module.exports = {
        pluginOptions: {
        electronBuilder: {
            builderOptions: {
                productName: "News App",
                appId: 'test.com',
                win: {
                    "target": [
                        "nsis"
                    ],
                  icon: 'public/svg.png',
                  "requestedExecutionLevel": "requireAdministrator"
                },
                "nsis": {
                    "installerIcon": "public/favicon.ico",
                    "uninstallerIcon": "public/favicon.ico",
                    "uninstallDisplayName": "CPU Monitor",
                    "license": "license.txt",
                    "oneClick": false,
                    "allowToChangeInstallationDirectory": true
                }
            },
        },
    },
    }
```

Electron 只在 路由模式 为 hash 时，才可以正常运行。否则，就会无法找到匹配的路径，故而选择 path: * 对应的路由；

如果你的 router 文件中，设置了 path: * ，那么 首页，就是path: * 对应的界面，比如刚才的首页弹出 404 ；而如果没有设置 path: * ，那么，就不出现首页blank！
————————————————
版权声明：本文为CSDN博主「RaySunWHUT」的原创文章，遵循CC 4.0 BY-SA版权协议，转载请附上原文出处链接及本声明。
原文链接：https://blog.csdn.net/qq_40994260/article/details/107440478

## 使用npm run electron:serve 的时候有tool不能控制的情况

为软件名称不对应导致的问题，在packge.json中修改。

## electron缓冲保存文件

1. 在~/.config中 ubuntu里面

2. 在C:\Users\xxxx\AppData\Roaming
