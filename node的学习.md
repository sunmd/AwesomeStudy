# node的学习

## npm将软件安装到哪里

```bash
npm install lodash
```

软件包会被安装到当前文件树中的 `node_modules` 子文件夹下

在这种情况下，`npm` 还会在当前文件夹中存在的 `package.json` 文件的 `dependencies` 属性中添加 `lodash` 条目。

使用 `-g` 标志可以执行全局安装：

```bash
npm install -g lodash
```

`npm root -g` 命令会告知其在计算机上的确切位置



### package.json 文件支持一种用于指定命令行任务（可通过使用以下方式运行）的格式：

使用此特性运行 Webpack 是很常见的：

```json
{
  "scripts": {
    "watch": "webpack --watch --progress --colors --config webpack.conf.js",
    "dev": "webpack --progress --colors --config webpack.conf.js",
    "prod": "NODE_ENV=production webpack -p --config webpack.conf.js",
  },
}
```

因此可以运行如下，而不是输入那些容易忘记或输入错误的长命令：

```console
$ npm run watch
$ npm run dev
$ npm run prod
```

## package.json 指南

`package.json` 文件中的内容没有固定的要求。 唯一的要求是必须遵守 JSON 格式，否则，尝试以编程的方式访问其属性的程序则无法读取它。

- `name` 设置了应用程序/软件包的名称。
- `version` 表明了当前的版本。
- `description` 是应用程序/软件包的简短描述。
- `main` 设置了应用程序的入口点。
- `private` 如果设置为 `true`，则可以防止应用程序/软件包被意外地发布到 `npm`。
- `scripts` 定义了一组可以运行的 node 脚本。
- `dependencies` 设置了作为依赖安装的 `npm` 软件包的列表。
- `devDependencies` 设置了作为开发依赖安装的 `npm` 软件包的列表。
- `engines` 设置了此软件包/应用程序在哪个版本的 Node.js 上运行。
- `browserslist` 用于告知要支持哪些浏览器（及其版本）。

以上所有的这些属性都可被 `npm` 或其他工具使用。



### repository

此属性指定了此程序包仓库所在的位置。

示例：

```json
"repository": "github:nodejscn/node-api-cn",
```

注意 `github` 前缀。 其他流行的服务商还包括：

```json
"repository": "gitlab:nodejscn/node-api-cn",
"repository": "bitbucket:nodejscn/node-api-cn",
```

可以显式地设置版本控制系统：

```json
"repository": {
  "type": "git",
  "url": "https://github.com/nodejscn/node-api-cn.git"
}
```

也可以使用其他的版本控制系统：

```json
"repository": {
  "type": "svn",
  "url": "..."
}
```

### main

设置软件包的入口点。

当在应用程序中导入此软件包时，应用程序会在该位置搜索模块的导出。

示例：

```json
"main": "src/main.js"
```

### private

如果设置为 `true`，则可以防止应用程序/软件包被意外发布到 `npm` 上。



### scripts

可以定义一组可以运行的 node 脚本。

示例：

```json
"scripts": {
  "dev": "webpack-dev-server --inline --progress --config build/webpack.dev.conf.js",
  "start": "npm run dev",
  "unit": "jest --config test/unit/jest.conf.js --coverage",
  "test": "npm run unit",
  "lint": "eslint --ext .js,.vue src test/unit",
  "build": "node build/build.js"
}
```

这些脚本是命令行应用程序。 可以通过调用 `npm run XXXX` 或 `yarn XXXX` 来运行它们，其中 `XXXX` 是命令的名称。 例如：`npm run dev`。

### dependencies

设置作为依赖安装的 `npm` 软件包的列表。

当使用 npm 或 yarn 安装软件包时：

```sh
npm install <PACKAGENAME>
yarn add <PACKAGENAME>
```

该软件包会被自动地插入此列表中。

示例：

```json
"dependencies": {
  "vue": "^2.5.2"
}
```

### devDependencies

设置作为开发依赖安装的 `npm` 软件包的列表。

它们不同于 `dependencies`，因为它们只需安装在开发机器上，而无需在生产环境中运行代码。

当使用 npm 或 yarn 安装软件包时：

```sh
npm install --dev <PACKAGENAME>
yarn add --dev <PACKAGENAME>
```

该软件包会被自动地插入此列表中。

示例：

```json
"devDependencies": {
  "autoprefixer": "^7.1.2",
  "babel-core": "^6.22.1"
}
```

### engines

设置此软件包/应用程序要运行的 Node.js 或其他命令的版本。

示例：

```json
"engines": {
  "node": ">= 6.0.0",
  "npm": ">= 3.0.0",
  "yarn": "^0.13.0"
}
```

### browserslist

用于告知要支持哪些浏览器（及其版本）。 Babel、Autoprefixer 和其他工具会用到它，以将所需的 polyfill 和 fallback 添加到目标浏览器。

示例：

```json
"browserslist": [
  "> 1%",
  "last 2 versions",
  "not ie <= 8"
]
```

此配置意味着需要支持使用率超过 1％（来自 [CanIUse.com](https://caniuse.com/) 的统计信息）的所有浏览器的最新的 2 个主版本，但不含 IE8 及更低的版本。

https://www.npmjs.com/package/browserslist

### 命令特有的属性

`package.json` 文件还可以承载命令特有的配置，例如 Babel、ESLint 等。

每个都有特有的属性，例如 `eslintConfig`、`babel` 等。 它们是命令特有的，可以在相应的命令/项目文档中找到如何使用它们。

## 软件包版本

在上面的描述中，已经看到类似以下的版本号：`〜3.0.0` 或 `^0.13.0`。 它们是什么意思，还可以使用哪些其他的版本说明符？

该符号指定了软件包能从该依赖接受的更新。

鉴于使用了 semver（语义版本控制），所有的版本都有 3 个数字，第一个是主版本，第二个是次版本，第三个是补丁版本，具有以下规则：

- `~`: 如果写入的是 `〜0.13.0`，则只更新补丁版本：即 `0.13.1` 可以，但 `0.14.0` 不可以。
- `^`: 如果写入的是 `^0.13.0`，则要更新补丁版本和次版本：即 `0.13.1`、`0.14.0`、依此类推。
- `*`: 如果写入的是 `*`，则表示接受所有的更新，包括主版本升级。
- `>`: 接受高于指定版本的任何版本。
- `>=`: 接受等于或高于指定版本的任何版本。
- `<=`: 接受等于或低于指定版本的任何版本。
- `<`: 接受低于指定版本的任何版本。

还有其他的规则：

- 无符号: 仅接受指定的特定版本。
- `latest`: 使用可用的最新版本。

还可以在范围内组合以上大部分内容，例如：`1.0.0 || >=1.1.0 <1.2.0`，即使用 1.0.0 或从 1.1.0 开始但低于 1.2.0 的版本。

## package-lock.json 文件

该文件旨在跟踪被安装的每个软件包的确切版本，以便产品可以以相同的方式被 100％ 复制（即使软件包的维护者更新了软件包）。

这解决了 `package.json` 一直尚未解决的特殊问题。 在 package.json 中，可以使用 semver 表示法设置要升级到的版本（补丁版本或次版本），例如：

- 如果写入的是 `〜0.13.0`，则只更新补丁版本：即 `0.13.1` 可以，但 `0.14.0` 不可以。
- 如果写入的是 `^0.13.0`，则要更新补丁版本和次版本：即 `0.13.1`、`0.14.0`、依此类推。
- 如果写入的是 `0.13.0`，则始终使用确切的版本。

无需将 node_modules 文件夹（该文件夹通常很大）提交到 Git，当尝试使用 `npm install` 命令在另一台机器上复制项目时，如果指定了 `〜` 语法并且软件包发布了补丁版本，则该软件包会被安装。 `^` 和次版本也一样。

> 如果指定确切的版本，例如示例中的 `0.13.0`，则不会受到此问题的影响。

可能是你，或者是其他人，会在某处尝试通过运行 `npm install` 初始化项目。

因此，原始的项目和新初始化的项目实际上是不同的。 即使补丁版本或次版本不应该引入重大的更改，但还是可能引入缺陷。

`package-lock.json` 会固化当前安装的每个软件包的版本，当运行 `npm install`时，`npm` 会使用这些确切的版本。

这个概念并不新鲜，其他编程语言的软件包管理器（例如 PHP 中的 Composer）使用类似的系统已有多年。

`package-lock.json` 文件需要被提交到 Git 仓库，以便被其他人获取（如果项目是公开的或有合作者，或者将 Git 作为部署源）。

当运行 `npm update` 时，`package-lock.json` 文件中的依赖的版本会被更新。



若要查看所有已安装的 npm 软件包（包括它们的依赖包）的最新版本，则：

```sh
npm list
```

例如：

```txt
❯ npm list
/Users/joe/dev/node/cowsay
└─┬ cowsay@1.3.1
  ├── get-stdin@5.0.1
  ├─┬ optimist@0.6.1
  │ ├── minimist@0.0.10
  │ └── wordwrap@0.0.3
  ├─┬ string-width@2.1.1
  │ ├── is-fullwidth-code-point@2.0.0
  │ └─┬ strip-ansi@4.0.0
  │   └── ansi-regex@3.0.0
  └── strip-eof@1.0.0
```

也可以打开 `package-lock.json` 文件，但这需要进行一些视觉扫描。

`npm list -g` 也一样，但适用于全局安装的软件包。

若要仅获取顶层的软件包（基本上就是告诉 npm 要安装并在 `package.json` 中列出的软件包），则运行 `npm list --depth=0`：

# 安装 npm 包的旧版本

可以使用 `@` 语法来安装 npm 软件包的旧版本：

```sh
npm install <package>@<version>
```

示例：

```sh
npm install cowsay
```

以上命令会安装 1.3.1 版本（在撰写本文时）。

使用以下命令可以安装 1.2.0 版本：

```sh
npm install cowsay@1.2.0
```

全局的软件包也可以这样做：

```sh
npm install -g webpack@4.16.4
```

可能还有需要列出软件包所有的以前的版本。 可以使用 `npm view <package> versions`：

```txt
❯ npm view cowsay versions

[ '1.0.0',
  '1.0.1',
  '1.0.2',
  '1.0.3',
  '1.1.0',
  '1.1.1',
  '1.1.2',
  '1.1.3',
  '1.1.4',
  '1.1.5',
  '1.1.6',
  '1.1.7',
  '1.1.8',
  '1.1.9',
  '1.2.0',
  '1.2.1',
  '1.3.0',
  '1.3.1' ]
```



# 将所有 Node.js 依赖包更新到最新版本

若要发觉软件包的新版本，则运行 `npm outdated`。

若要将所有软件包更新到新的主版本，则全局地安装 `npm-check-updates` 软件包：

```sh
npm install -g npm-check-updates
```

然后运行：

```sh
ncu -u
```

这会升级 `package.json` 文件的 `dependencies` 和 `devDependencies` 中的所有版本，以便 npm 可以安装新的主版本。

现在可以运行更新了：

```sh
npm update
```

如果只是下载了项目还没有 `node_modules` 依赖包，并且想先安装新的版本，则运行：

```sh
npm install
```