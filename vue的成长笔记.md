# vue的成长笔记

## 疑难杂症

###  3:12  warning  Insert `·`  prettier/prettier

意思是`<Header/>` 写成<Header />

可以使用`npm run lint --fix`自动进行代码格式化

在visiocode中可以用

安装 过Prettier插件 Prettier - Code formatter，每次修改代码后都要手动格式化代码很麻烦，可以在系统设置中增加`"editor.formatOnSave": true`

.eslintrc.js

参考地址:

https://www.it610.com/article/1279604537547571200.htm

### router.beforeEach( TypeError: next is not a function

```vue
router.beforeEach((to, from, next) => {
  NProgress.start();
  next();
});
// 必须填写三个参数,不是可以进行提取,也学可以用对象来提取,后面进行测试一下
```

### Error: ENOSPC: System limit for number of file watchers reached, watch

　当前问题主要是因为gulp的watch需要监听很多文件的改动，但是fedora、ubuntu系统的文件句柄其实是有限制的，因此可以使用以下命令： 
 　　echo fs.inotify.max_user_watches=524288 | sudo tee -a /etc/sysctl.conf && sudo sysctl -p 
 　　对于以上Linux下gulp报错Error：watch ENOSPC的解决方法就介绍完了，如果用户也出现以上同样问题，那么可以按照操作方法赶紧试试解决吧！

### 解决vue项目Vue-router 报NavigationDuplicated: "Navigating to current location ("") is not allowed"的问题

#### 出现问题的原因

其原因在于Vue-router在3.1之后把$router.push()方法改为了Promise。所以假如没有回调函数，错误信息就会交给全局的路由错误处理，因此就会报上述的错误。

vue-router是先报了一个Uncaught (in promise)的错误(push没加回调)，然后报NavigationDuplicated的错误(路由出现的错误，全局错误处理打印了出来。

#### 解决方法

方法一，有待思考原理

```javascript
import Router from 'vue-router'

const originalPush = Router.prototype.push
Router.prototype.push = function push(location, onResolve, onReject) {
  if (onResolve || onReject) return originalPush.call(this, location, onResolve, onReject)
  return originalPush.call(this, location).catch(err => err)
```

方法二，有待思考原理

```javascript
this.$router.push('/').catch(err => {err})
this.$router.replace('/').catch(err => {err})
```

As I remember well, you can use catch clause after this.$router.push. Then it will look like:

```
this.$router.push("/admin").catch(()=>{});
```

This allows you to only avoid the error displaying, because browser thinks the exception was handled.

*参考csdn和stackoverflow* 

https://stackoverflow.com/questions/62462276/how-to-solve-avoided-redundant-navigation-to-current-location-error-in-vue

https://blog.csdn.net/weixin_44727491/article/details/105278494

### [Vue warn]: You are using the runtime-only build of Vue where the template  compiler is not available. Either pre-compile the templates into render  functions, or use the compiler-included build.

```json
On the vue.config.js file put:

module.exports = {
runtimeCompiler: true
}
```

https://cli.vuejs.org/config/#runtimecompiler

### Object(…) is not a function报错解决(lodash/findLast)

```javascript
// 由于结构的方式出现了问题.不需要用{}来解析直接可以用
import { findLast } from "lodash/findLast";
// 改成这个
import findLast from "lodash/findLast";

```

