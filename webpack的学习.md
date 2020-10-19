# webpack的学习

## 遇到的问题

### Proxy error: Could not proxy request /api/dashboard/chart?ID=123456 from localhost:8080 to http://localhost:3000/.
See https://nodejs.org/api/errors.html#errors_common_system_errors for more information (ECONNREFUSED).

这个问题为vue.config.js上的配置错误,引用了devServer功能引起的

目前来看vue-cli->webpack-> http-proxy-middleware

在context matching上用的是Glob

对应的格式英文名称

```
         foo://example.com:8042/over/there?name=ferret#nose
         \_/   \______________/\_________/ \_________/ \__/
          |           |            |            |        |
       scheme     authority       path        query   fragment
```

使用了:

```js
module.exports = {
    devServer: {
        proxy: {
            "@(^/api)": {
                target: "http://loaclhost:3000"
            }
        }
    }
}
```

就解决了,但是过滤上依然把非api开头的显示了出来了