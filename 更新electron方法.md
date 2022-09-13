# 更新electron方法

## 官方推荐方法

最简单并且获得官方支持的方法是利用内置的[Squirrel](https://github.com/Squirrel)框架和Electron的[autoUpdater](https://www.electronjs.org/zh/docs/latest/api/auto-updater)模块。

[官方地址](https://www.electronjs.org/zh/docs/latest/tutorial/updates)

### 使用 `update.electronjs.org`

Electron 团队维护 [update.electronjs.org](https://github.com/electron/update.electronjs.org)，一个免费开源的网络服务，可以让 Electron 应用使用自动更新。 这个服务是设计给那些满足以下标准的 Electron 应用：

- 应用运行在 macOS 或者 Windows
- 应用有公开的 GitHub 仓库
- 编译的版本发布在 GitHub Releases
- 编译的版本已代码签名

默认情况下，这个模块会在应用启动的时候检查更新，然后每隔十分钟再检查一次。 当发现了一个更新，它会**自动在后台下载**。 当下载完成后，会显示一个对话框以允许用户重启应用。

目前只支持 macOS 和 Windows 版本。 在 Linux 上没有内置的自动更新程序，因此建议使用发行版包管理器来更新您的应用程序。

### Windows[​](https://www.electronjs.org/zh/docs/latest/api/auto-updater#windows "直接链接到标题")

在 Windows 上, 你必须将应用程序安装在用户的计算机上才能使用`autoUpdater`, 所以推荐使用 [electron-winstaller](https://github.com/electron/windows-installer), [electron-forge](https://github.com/electron-userland/electron-forge) 或者 [grunt-electron-installer](https://github.com/electron/grunt-electron-installer) 模块来生成Windows安装程序。

当使用 [electron-winstaller](https://github.com/electron/windows-installer) 或 [electron-forge](https://github.com/electron-userland/electron-forge) 时，请确保不要在[第一次运行](https://github.com/electron/windows-installer#handling-squirrel-events)你的应用程序时进行更新操作(也可查看这个 [issue获取更多信息](https://github.com/electron/electron/issues/7155))。 还建议使用 [electron-squirrel-startup](https://github.com/mongodb-js/electron-squirrel-startup) 来创建应用程序的桌面快捷方式。

容器许愿

https://github.com/ArekSredzki/electron-release-server

## electron-updater

我们在开发一款electron桌面应用中，我们希望只要我们发布了最新的版本，用户就会收到新包的更新提示。在electron自带的有`autoUpdater`，本次我们选择的是`electron-updater`这个第三方插件。先说一下electron自带的`autoUpdater`的缺点是没有办法控制什么时候下载(目前我看官方文档是没有这个功能，如果有欢迎指正)，且不知道下载的进度；而`electron-updater`却有这个功能，这个功能最大的好处就是我们可以在正式环境下存在多个版本，时时监听下载进度；

> ## GenericServerOptions[¶](https://www.electron.build/configuration/publish#genericserveroptions "Permanent link")
> 
> Generic (any HTTP(S) server) options. In all publish options [File Macros](https://www.electron.build/file-patterns#file-macros) are supported.
> 
> - **`provider`** “generic” - The provider. Must be `generic`.
> - **`url`** String - The base url. e.g. `https://bucket_name.s3.amazonaws.com`.
> - `channel` = `latest` String - The channel.
> - `useMultipleRangeRequest` Boolean - Whether to use multiple range requests for differential update. Defaults to `true` if `url` doesn’t contain `s3.amazonaws.com`.
> 
> Inherited from `PublishConfiguration`:
> 
> - `publishAutoUpdate` = `true` Boolean - Whether to publish auto update info files.
>   
>   Auto update relies only on the first provider in the list (you can specify several publishers). Thus, probably, there`s no need to upload the metadata files for the other configured providers. But by default will be uploaded.

https://www.electron.build/configuration/publish

https://nklayman.github.io/vue-cli-plugin-electron-builder/guide/recipes.html#table-of-contents

## 热更新

这样的话就实现了，渲染进程的压缩；在更新中就可以实现只下载bundle.zip就可以更新；以前需要下载大约174MB的文件，现在只需要下载18MB的文件，下载约节省了90%流量，大大提高用户的更新速度

### 1页面放在服务器上

如果你只是想用electron的壳，而不是想用electron中带有的node环境，拿就直接使用网站的更新就好了，

```js
win.mainWin.loadURL(winURL) // winURL 你网站的URL地址，
```

### 2打包下载综合

[electron 热更新的最优实践 - 掘金](https://juejin.cn/post/6903352556656230408#heading-1)

在这个流程图中可以发现有以下几个问题：

1. 渲染进程如何和主进程分开打包
2. 如何压缩渲染进程的文件成zip格式
3. 如何判断当前版本是热更新版本需要更新的版本；或者说热更新版本存放在哪个位置
4. 在主进程中如何解压渲染进程的zip文件，删除原有的渲染进程的文件夹
5. 更新完成后如何重启

作者：chenyuhu  
链接：https://juejin.cn/post/6903352556656230408  
来源：稀土掘金  
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。

### 3直接热更新asar

如果`electron-updater`支持差异更新，那应该是最佳选择了，貌似目前正在尝试，期待ing...

[(17条消息) 用于更新electron-app.asar的nodejs模块_weixin_34380948的博客-CSDN博客](https://blog.csdn.net/weixin_34380948/article/details/91381580)

#### 如何工作 (Read this first)

- 用于处理更新Electron应用程序内app.asar文件的过程;它只是用名为“update.asar”的新文件替换app.asar文件（在/ resources /）！
- 检查“更新”必须由应用程序触发。 `EAU`不会自行进行任何形式的定期检查。
- `EAU`与API进行对话，告诉它是否有新的更新。
  - API接收来自EAU的请求，其中包含客户端当前版本的应用程序（必须在应用程序package.json文件中指定）
  - 然后，API以新更新响应，或者只是false以中止。
  - 如果有可用更新，则API应使用此更新update.asar文件的源进行响应。
  - EAU然后下载.asar文件，删除旧的app.asar并将update.asar重命名为app.asar。

#### 为何要使用它 ? (用例)

- 如果您认为这些太复杂而无法实施: [www.npmjs.com/package/ele…](https://link.juejin.im/?target=https%3A%2F%2Fwww.npmjs.com%2Fpackage%2Felectron-updater) [electron.atom.io/docs/v0.33.…](https://link.juejin.im/?target=http%3A%2F%2Felectron.atom.io%2Fdocs%2Fv0.33.0%2Fapi%2Fauto-updater%2F)
- 如果您认为在更换一个文件（通常为40MB），.app或.exe文件（最多100MB）是不合理的。
- 需要在更新时查看进度。
- 选择使用服务器端检查或客户端检查。
- 可以使用zip压缩文件，压缩你的ASAR使其更小。

### 开源项目年久失修

如果调整有可能是好的方案。整理下载不到1MB，下载之后可以重启应用

## deepin编译

```shell
 INFO  Building app with electron-builder:
  • electron-builder  version=22.14.5 os=5.10.60-amd64-desktop
  • description is missed in the package.json  appPackageFile=/home/sunmd/WorkSpace/lixiang-brushwriting/dist_electron/bundled/package.json
  • author is missed in the package.json  appPackageFile=/home/sunmd/WorkSpace/lixiang-brushwriting/dist_electron/bundled/package.json
  • writing effective config  file=dist_electron/builder-effective-config.yaml
  • packaging       platform=linux arch=x64 electron=13.6.3 appOutDir=dist_electron/linux-unpacked
  • downloading     url=https://cdn.npm.taobao.org/dist/electron/13.6.3/electron-v13.6.3-linux-x64.zip size=78 MB parts=8
  • downloaded      url=https://cdn.npm.taobao.org/dist/electron/13.6.3/electron-v13.6.3-linux-x64.zip duration=3.966s
  • building        target=snap arch=x64 file=dist_electron/lixiang-brushwriting_0.1.0_amd64.snap
  • building        target=AppImage arch=x64 file=dist_electron/迪璞刷写机-0.1.0.AppImage
  • application Linux category is set to default "Utility"  reason=linux.category is not set and cannot map from macOS docs=https://www.electron.build/configuration/linux
  • default Electron icon is used  reason=application icon is not set
  • application Linux category is set to default "Utility"  reason=linux.category is not set and cannot map from macOS docs=https://www.electron.build/configuration/linux
 DONE  Build complete!
```

https://www.cnblogs.com/itqn/p/electron_autoupdate.html_
