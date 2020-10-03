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