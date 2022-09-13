# electron多窗口调研

## 多窗口实现原理

Electron inherits its multi-process architecture from Chromium, which makes the framework architecturally very similar to a modern web browser.

electron 从Chromium集成多进程，这种架构非常相似现代的网页浏览器

the Chrome team decided that each tab would render in its own process, limiting the harm that buggy or malicious code on a web page could cause to the app as a whole. A single browser process then controls these processes, as well as the application lifecycle as a whole.

Electron applications are structured very similarly. As an app developer, you control two types of processes: [main](https://www.electronjs.org/docs/latest/tutorial/process-model#the-main-process) and [renderer](https://www.electronjs.org/docs/latest/tutorial/process-model#the-renderer-process). These are analogous to Chrome's own browser and renderer processes outlined above.

Each Electron app spawns a separate renderer process for each open `BrowserWindow` (and each web embed). As its name implies, a renderer is responsible for *rendering* web content. For all intents and purposes, code ran in renderer processes should behave according to web standards (insofar as Chromium does, at least).

总体来说，是一个主进程，和多个独立的渲染进程

文献：[Process Model | Electron (electronjs.org)](https://www.electronjs.org/docs/latest/tutorial/process-model#window-management)
