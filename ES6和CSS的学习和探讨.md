# ES6和CSS 学习和探讨

## ES6模块加载和CommonJs加载

[js] File is a CommonJS module; it may be converted to an ES6 module.



ES6模块加载的机制，与CommonJS模块完全不同。CommonJS模块输出的是一个值的拷贝，而ES6模块输出的是值的引用。

ES6模块的运行机制与CommonJS不一样，它遇到模块加载命令`import`时，不会去执行模块，而是只生成一个动态的只读引用。等到真的需要用到时，再到模块里面去取值，换句话说，ES6的输入有点像Unix系统的“符号连接”，原始值变了，`import`输入的值也会跟着变。因此，ES6模块是动态引用，并且不会缓存值，模块里面的变量绑定其所在的模块。

CommonJS的一个模块，就是一个脚本文件。`require`命令第一次加载该脚本，就会执行整个脚本，然后在内存生成一个对象。



引用

https://github.com/microsoft/vscode/issues/47299

http://caibaojian.com/es6/module.html