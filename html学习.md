### 对象的三种类型

- 用户自定义对象(user-defined object)
- 内建对象(native object)
- 宿主对象(host object)

id和class都可以被getElementById或这css所引用,返回的值可以用typeof操作符来查看结果

文档中的每一个元素都是一个对象

### 一份文档就是一颗节点数

### 节点的单三类型

- 对象节点

  - getElementById
  - getElementByTagName
  - getElementByClassName

- 属性节点

  - setAttribute

    设置属性之后,在浏览器的view source 上不能看到对应的修改

    DOM的工作模式:先加载文档的静态内容,在动态刷新. 动态刷新不影响静态内容,对页面内容进行刷新却不需要再浏览器刷新页面

  - getAttribute

- 文本节点