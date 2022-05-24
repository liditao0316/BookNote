## 一、npm简介

`npm` 全称为 `Node Package Manager`，是一个基于 `Node.js` 的包管理器，也是整个 `Node.js` 社区最流行、支持的第三方模块最多的包管理器。npm的初衷：JavaScript开发人员更容易分享和重用代码。

1. nodejs = ECMAScript + 核心模块
2. 自己遵循 commonjs 规范写出模块，如果写的是功能模块（日期处理datejs，数字处理numberjs）。如果可以把这些模块分享出来，以后谁要进行相关功能开发的时候，直接拿开发好的模块使用即可，没必要自己在开发。在互联网有一个网站专门收集这样的工具包。https://www.npmjs.cn/。
3. 如果我们要使用这个网站里面的包，则我们需要使用一个功能，叫做 npm。

官网：https://www.npmjs.cn/

<https://www.npmjs.com/package/md5> 

npm可以用来：

- 允许用户获取第三方包并使用
- 允许用户将自己编写的包或命令行程序进行发布分享

**npm安装：**

`npm`不需要单独安装。在安装 `Node` 的时候，会连带一起安装`npm`。

执行下面的命令可以用来查看本地安装的 npm 的版本号。

```bash
npm -v
```

如果想升级 npm ，可以这样

```bash
npm install npm --global
```

```js
NPM是随同NodeJS一起安装的包管理工具
npm -v
npm install npm -g

npm i  express
npm install express      # 本地安装
npm install express -g   # 全局安装
npm list -g
===============================
npm uninstall express
npm update express  #更新模块

npm init   #创建模块，package.json
npm init  -y
```





```js
Package.json 属性说明
name  #包名。

version #包的版本号。

description #包的描述。

homepage #包的官网 url 。

author #包的作者姓名。

contributors  #包的其他贡献者姓名。

dependencies #依赖包列表。如果依赖包没有安装，npm 会自动将依赖包安装在 node_module 目录下。

repository #包代码存放的地方的类型，可以是 git 或 svn，git 可在 Github 上。

main  #main 字段指定了程序的主入口文件，require('moduleName') 就会加载这个文件。这个字段的默认值是模块根目录下面的 index.js。

keywords #关键字
```





## 二、npm体验

以安装和使用md5模块为例：

1、项目目录下，执行命令 npm init，目录下会多一个package.json文件(这个文件1、记录项目相关信息，如项目名称，项目版本2、后期会记录项目中使用的第三方模块)

2、项目目录下，执行命令 npm install md5，这时候就会开始联网下载md5这个包，下载过程需要耐心等待，等待时间视网速而定。

下载完后：本地项目目录下多了一个node_modules文件夹，我们刚才所下载的md5包及其相关依赖包都在这个文件夹里面了。以后我们开发中需要下载其他包，都会在下载在这个文件夹中。

4、下载完就可以在项目中去导入然后使用了：

```js
var md5 = require('md5');
console.log(md5("12345789"));
```

运行就会得到一个加密字符串。



## 三、小练习

实现一个，数字转大写的功能      如：  123    转   壹佰贰拾叁

在  https://www.npmjs.com  上搜索功能关键字

找对应可能用上的包，参考文档，进行安装，使用

#### 为什么要使用 nodemon

```
在编写调试 Node.js 项目的时候，如果修改了项目的代码，则需要频繁的手动 close 掉，然后再重新启动，非常繁琐。
现在，我们可以使用 nodemon（https://www.npmjs.com/package/nodemon） 这个工具，它能够监听项目文件的变动，当代码被修改后，nodemon 会自动帮我们重启项目，极大方便了开发和调试。
```

#### 安装 nodemon

在终端中，运行如下命令，即可将 nodemon 安装为全局可用的工具：

```
npm i -g nodemon
```

#### 使用 nodemon

```
当基于 Node.js 编写了一个网站应用的时候，传统的方式，是运行 node app.js 命令，来启动项目。这样做的坏处是：代码被修改之后，需要手动重启项目。
现在，我们可以将 node 命令替换为 nodemon 命令，使用 nodemon app.js 来启动项目。这样做的好处是：代码被修改之后，会被 nodemon 监听到，从而实现自动重启项目的效果。

```





## 四、nodemon包的使用

我们前面使用node的http模块书写过web服务器，但是每次改写一点代码都需要重启服务器，开发不是很方便。nodemon可以监听代码的改动自动更新，不需要重启服务器程序就可以看效果。

文档：<https://www.npmjs.com/package/nodemon> 

下载：npm i  -g nodemon 

说明：  -g  表示安装在全局，  这种安装方式不同于前面的安装，它只需要安装一次，就能一直使用。安装的时候会有一个专门的安装目录（安装完成会有提示安装位置，如果忘记了，可以通过npm root -g命令查看安装在哪里）

安装成功，项目目录下，通过命令**nodemon index.js**启动服务器即可。





## 五、nrm安装与配置

1.nrm

nrm(npm registry manager )是npm的镜像源管理工具，有时候国外资源太慢，使用这个就可以快速地在 npm 源间切换

2.安装nrm

在命令行执行命令，npm install -g nrm，全局安装nrm。

3.使用

执行命令nrm ls查看可选的源。

```
nrm ls                                                                                                                                  
```

4.切换

如果要切换到taobao源，执行命令

```
nrm use taobao
```

