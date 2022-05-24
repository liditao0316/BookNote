## 一、Express 简介

#### 1.1 什么是 Express

```js
官方给出的概念：Express 是基于 Node.js 平台，快速、开放、极简的 Web 开发框架。
通俗的理解：Express 的作用和 Node.js 内置的 http 模块类似，是专门用来创建 Web 服务器的。
Express 的本质：就是一个 npm 上的第三方包，提供了快速创建 Web 服务器的便捷方法。
Express 的中文官网： http://www.expressjs.com.cn/
```

#### 1.2 Express 能做什么

对于前端程序员来说，最常见的两种服务器，分别是：
 **1) Web 网站服务器：专门对外提供 Web 网页资源的服务器。**
 **2) API 接口服务器：专门对外提供 API 接口的服务器。**
使用 Express，我们可以方便、快速的创建 Web 网站的服务器或 API 接口的服务器。

#### 1.3 安装

在项目所处的目录中，运行如下的终端命令，即可将 express 安装到项目中使用

```
npm i express@4.17.1
```

Express 的基本使用

## 二、Express入门

#### 2.1 创建基本的 Web服务器

```js
//导入express
const express = require('express')
//创建web服务器
const app = express()
app.use(express.urlencoded({extended:false}))
//调用app.listen（端口号，启动成功后回调函数）,启动服务器
app.listen(80, () => {
  console.log('服务器已启动')
})
```

#### 2.2 监听 GET 请求

通过 app.get() 方法，可以监听客户端的 GET 请求

#### 2.3 监听 POST 请求

通过 app.post() 方法，可以监听客户端的 POST 请求

#### 2.4 把内容响应给客户端

通过 res.send() 方法，可以把处理好的内容，发送给客户端：

```js
//参数1：客户端请求的URL地址
//参数2:请求对应的处理函数
app.get('/user', function (req, res) {
  //req:请求对象（包含了与请求相关的属性与方法)request请求
  //res:响应对象(包含了与响应相关的属性与方法)response响应
  //向客户端发送json对象
  res.send({ uname: '小张', age: 20, gender: '男' })
})
app.post('/user', function (req, res) {
  	//向客户端发送文本信息
  res.send('post')
})
```

## 三、URL

#### 3.1 获取 URL中携带的查询参数

通过 **req.query** 对象，可以访问到客户端通过查询字符串的形式，发送到服务器的参数：

```js
app.get('/', function (req, res) {
  //req.query默认是一个空对象
  //客户端使用？uname=zs&age=20这种查询字符串形式，发送服务器的参数
  //可以通过req.query对象访问到
  //req.query.uname    req.query.age
  res.send(`${req.query.uname}==========${req.query.age}`)
})
```

#### 3.2 获取 URL中的动态参数

通过 req.params 对象，可以访问到 URL 中，通过 : 匹配到的动态参数：

```js
//url地址中，可以通过:参数名的形式，匹配动态参数值
app.get('/user/:id/:age', function (req, res) {
  // req.params默认是一个空对象
  //里面存放通过 ：动态匹配到的参数值
  res.send(`${req.params.id}========${req.params.age}`)
})
```

## 四、托管静态资源

#### 4.1 express.static()

express 提供了一个非常好用的函数，叫做 express.static()，通过它，我们可以非常方便地创建一个静态资源服务器，例如，通过如下代码就可以将 public 目录下的图片、CSS 文件、JavaScript 文件对外开放访问了：

```js
app.use(express.static('public'))
现在，你就可以访问 public 目录中的所有文件了：
// http://localhost:3000/images/01.jpg
// http://localhost:3000/css/style.css
// http://localhost:3000/js/login.js
注意：Express 在指定的静态目录中查找文件，并对外提供资源的访问路径。因此，存放静态文件的目录名不会出现在 URL 中。
```

#### 4.2 托管多个静态资源目录

如果要托管多个静态资源目录，请多次调用 express.static() 函数

```js
app.use(express.static('public'))
app.use(express.static('files'))
访问静态资源文件时，express.static() 函数会根据目录的添加顺序查找所需的文件。
```

#### 4.3 挂载路径前缀

如果希望在托管的静态资源访问路径之前，挂载路径前缀，则可以使用如下的方式：

```js
app.use("/public",express.static('public'))
现在，你就可以通过带有 /public 前缀地址来访问 public 目录中的文件了：
// http://localhost:3000/public/images/01.jpg
// http://localhost:3000/public/css/style.css
// http://localhost:3000/public/js/app.js
```

## 五、路由的概念

#### 5.1 什么是路由

```
广义上来讲，路由就是映射关系。路径对应页面
```

#### 5.2 Express 中的路由

在 Express 中，路由指的是客户端的请求与服务器处理函数之间的**映射**关系。

Express 中的路由分 3 部分组成，分别是请求的类型、请求的 URL 地址、处理函数，格式如下：

```
app.methods(path,handle)
```

Express 中的路由

```js
//匹配GET请求，且请求URL为/
app.get( '/', function (req,res) {res.send( "get ')
)
                                  
//匹配POST请求，且请求URL为/
app.post( '/', function (req,res) {res.send( 'post' )})
```

最简单的用法

在 Express 中使用路由最简单的方式，就是把路由挂载到app 上,如下：

```js
const express = require( 'express " )
//创建web服务器，命名为app
const app = express()
//挂载路由
app.get( "/", (req，res) => {res.send( 'get" )
app.post( "/",(req，res) =>{ res.send( 'Post')})
//启动web服务器
app.listen(80，() =>{ console.log( '服务器已启动'))
```

####  5.3 模块化路由

为了方便对路由进行模块化的管理，Express 不建议将路由直接挂载到 app 上，而是推荐将路由抽离为单独的模块。
将路由抽离为单独模块的步骤如下：
**1）创建路由模块对应的 .js 文件**
**2）调用 express.Router() 函数创建路由对象
3）向路由对象上挂载具体的路由**
4**）使用 module.exports 向外共享路由对象**
**5）使用 app.use() 函数注册路由模块**

#### 5.4 创建路由模块

```js
var express = require('express') // 1．导入express
var router = express.Router() // 2．创建路由对象
router.get( '/user/list'，function (req，res) {res.send( '列表文件')}//3．挂载获取用户列表的路由
router.post('/user/add ',function (req，res) {res.send( '增加用户')} // 4．挂载添加用户的路由
module.exports = router  //向外导出路由对象
```

#### 5.5 注册路由模块

```js
// 1．导入路由模块
const userRouter = require( ' ./router/user.js ')
// 2．使用app.use()注册路由模块
app.use(userRouter)
```

#### 5.6 为路由模块添加前缀

类似于托管静态资源时，为静态资源统一挂载访问前缀一样，路由模块添加前缀的方式也非常简单：

```js
// 1．导入路由模块
const userRouter = require( './router/user.js ' )
// 2．使用app.use()注册路由模块
app.use(userRouter)
//添加统一的访问前缀
app.use( 'api', userRouter)
```



## 六、中间件的概念

#### 6.1 什么是中间件

**中间件（Middleware ），特指业务流程的中间处理环节。**

#### 6.2 Express 中间件的调用流程

```
当一个请求到达 Express 的服务器之后，可以连续调用多个中间件，从而对这次请求进行预处理。
```

![](4、express和中间件.assets/15.png)

#### 6.3 Express 中间件的格式

Express 的中间件，本质上就是一个 function 处理函数，Express 中间件的格式如下：

```js
app.get("/",function(req,res,next){
	next()
})
```

```
注意：中间件函数的形参列表中，必须包含 next 参数。而路由处理函数中只包含 req 和 res。
```

#### 6.4 next 函数的作用

```
next 函数是实现多个中间件连续调用的关键，它表示把流转关系转交给下一个中间件或路由。
```

![](4、express和中间件.assets/17.png)

#### 6.5 中间件的作用

```
多个中间件之间，共享同一份 req 和 res。基于这样的特性，我们可以在上游的中间件中，统一为 req 或 res 对象添加自定义的属性或方法，供下游的中间件或路由进行使用。
```

```js
const express = require('express')
const app = express()
// 这是定义全局中间件的简化形式
app.use((req, res, next) => {
  // 获取到请求到达服务器的时间
  const time = Date.now()
  // 为 req 对象，挂载自定义属性，从而把时间共享给后面的所有路由
  req.startTime = time
  next()
})
app.get('/', (req, res) => {
  res.send('首页.' + req.startTime)
})
app.get('/user', (req, res) => {
  res.send('用户页' + req.startTime)
})
app.listen(80, () => {
  console.log('http://127.0.0.1')
})
```

#### 6.6 定义中间件函数

```js
const express = require('express')
const app = express()
//定义全局中间件,客户端发起的任何请求，到达服务器之后，都会触发的中间件，叫做全局生效的中间件。
const mw=function(req,res,next){
	console.log("这是中间件")
	next()
}
//注册中间件,通过调用 app.use(中间件函数)，即可定义一个全局生效的中间件
app.use(mw)
=====================
//简化版
app.use(function(req,res,next){
    console.log("这是中间件")
    next()
})


app.get('/', (req, res) => {
  res.send('首页')
})

app.get('/user', (req, res) => {
  res.send('用户页')
})

app.listen(80, () => {
 console.log('服务器已启动')
})
```

#### 6.7 定义多个全局中间件

```
可以使用 app.use() 连续定义多个全局中间件。客户端请求到达服务器之后，会按照中间件定义的先后顺序依次进行调用
```

```js
app.use(function(req,res,next){
	console.log("调用第一个中间件")
	next()
})
app.use(function(req,res,next){
	console.log("调用第二个中间件")
	next()
})
//调用这个路由，会依次调用这两个中间
app.get("/login",(req,res)=>{
    res.send("登陆页面")
})
```

#### 6.8 局部生效的中间件

```
不使用 app.use() 定义的中间件，叫做局部生效的中间件
```

```js
// 导入 express 模块
const express = require('express')
// 创建 express 的服务器实例
const app = express()
// 1. 定义中间件函数
const mw1 = (req, res, next) => {
  console.log('调用了局部生效的中间件')
  next()
}
// 2. 创建路由
app.get('/', mw1, (req, res) => {
  res.send('Home page.')
})
app.get('/user', (req, res) => {
  res.send('User page.')
})

// 调用 app.listen 方法，指定端口号并启动web服务器
app.listen(80, function () {
  console.log('服务器已启动')
})
```

#### 6.9 定义多个局部中间件

可以在路由中，通过如下两种等价的方式，使用多个局部中间件：

```js
//以下两种写法是"完全等价"的，可根据自己的喜好，选择任意一种方式进行使用
app.get( '/',mw1,mw2,(req，res) => { res.send("hello")})
app.get( '/',[mw1,mw2],(req,res) => { res.send( 'hello')})
```

**使用中间件的5个使用注意事项**

**1）一定要在路由之前注册中间件**
**2）客户端发送过来的请求，可以连续调用多个中间件进行处理**
**3）执行完中间件的业务代码之后，不要忘记调用 next() 函数**
**4）为了防止代码逻辑混乱，调用 next() 函数后不要再写额外的代码**
**5）连续调用多个中间件时，多个中间件之间，共享 req 和 res 对象**

- **定义多个全局中间件**

```js
const express = require('express')
const app = express()
// 定义第一个全局中间件
app.use((req, res, next) => {
  console.log('调用了第1个全局中间件')
  next()
})
// 定义第二个全局中间件
app.use((req, res, next) => {
  console.log('调用了第2个全局中间件')
  next()
})

// 定义一个路由
app.get('/user', (req, res) => {
  res.send('User page.')
})

app.listen(80, () => {
  console.log('服务器已启动')
})
```

- **同时使用多个局部中间件**

```js
// 导入 express 模块
const express = require('express')
// 创建 express 的服务器实例
const app = express()
// 1. 定义中间件函数
const mw1 = (req, res, next) => {
  console.log('调用了第一个局部生效的中间件')
  next()
}
const mw2 = (req, res, next) => {
  console.log('调用了第二个局部生效的中间件')
  next()
}
// 2. 创建路由
app.get('/', [mw1, mw2], (req, res) => {
  res.send('Home page.')
})
app.get('/user', (req, res) => {
  res.send('User page.')
})
// 调用 app.listen 方法，指定端口号并启动web服务器
app.listen(80, function () {
  console.log('Express server running at http://127.0.0.1')
})
```

#### 6.10 中间件的分类

为了方便大家理解和记忆中间件的使用，Express 官方把常见的中间件用法，分成了 5 大类，分别是：
 **1)应用级别的中间件**
 **2)路由级别的中间件**
 **3)错误级别的中间件**
 **4)Express 内置的中间件**
 **5)第三方的中间件**

##### 1、应用级别的中间件

```
通过 app.use() 或 app.get() 或 app.post() ，绑定到 app 实例上的中间件，叫做应用级别的中间件：
```

```js
//应用级别的中间件（全局中间件)
app.use((req,res, next) =>{
    next()
}
//应用级别的中间件（局部中间件)
app.get( '/',mw1,(req,res) =>{
	res.send( ' Home page. ')
}
```

##### 2、路由级别的中间件

绑定到 express.Router() 实例上的中间件，叫做路由级别的中间件。它的用法和应用级别中间件没有任何区别。只不过，应用级别中间件是绑定到 app 实例上，路由级别中间件绑定到 router 实例上

```js
var app = express()
var router = express.Router()
//路由级别的中间件
router.use(function(req,res,next) {
	console.log( 'Time: ' ,Date.now())
	next()
})
app.use("/",router)
```

##### 3、错误级别的中间件

```js
错误级别中间件的作用：专门用来捕获整个项目中发生的异常错误，从而防止项目异常崩溃的问题。
// 格式：错误级别中间件的 function 处理函数中，必须有 4 个形参，形参顺序从前到后，分别是 (err, req, res, next)。
```

![](4、express和中间件.assets/27.png)

```js
// 注意：错误级别的中间件，必须注册在所有路由之后！
```

错误级别中间件的使用

```js
// 导入 express 模块
const express = require('express')
// 创建 express 的服务器实例
const app = express()
// 1. 定义路由
app.get('/', (req, res) => {
  // 1.1 人为的制造错误
  throw new Error('服务器内部发生了错误！')
  res.send('Home page.')
})
// 2. 定义错误级别的中间件，捕获整个项目的异常错误，从而防止程序的崩溃
app.use((err, req, res, next) => {
  console.log('发生了错误！' + err.message)
  res.send('Error：' + err.message)
})
// 调用 app.listen 方法，指定端口号并启动web服务器
app.listen(80, function () {
  console.log('Express server running at http://127.0.0.1')
})
```

##### 4、Express内置的中间件

```js
自 Express 4.16.0 版本开始，Express 内置了 3 个常用的中间件，极大的提高了 Express 项目的开发效率和体验：
 // express.static 快速托管静态资源的内置中间件，例如： HTML 文件、图片、CSS 样式等（无兼容性）
 // express.json 解析 JSON 格式的请求体数据（有兼容性，仅在 4.16.0+ 版本中可用）
 // express.urlencoded 解析 URL-encoded 格式的请求体数据（有兼容性，仅在 4.16.0+ 版本中可用）
```

![](4、express和中间件.assets/28.png)

```js
// 导入 express 模块
const express = require('express')
// 创建 express 的服务器实例
const app = express()
// 注意：除了错误级别的中间件，其他的中间件，必须在路由之前进行配置
// 通过 express.json() 这个中间件，解析表单中的 JSON 格式的数据
app.use(express.json())
// 通过 express.urlencoded() 这个中间件，来解析 表单中的 url-encoded 格式的数据
app.use(express.urlencoded({ extended: false }))
app.post('/user', (req, res) => {
  // 在服务器，可以使用 req.body 这个属性，来接收客户端发送过来的请求体数据
  // 默认情况下，如果不配置解析表单数据的中间件，则 req.body 默认等于 undefined
  console.log(req.body)
  res.send('ok')
})

app.post('/book', (req, res) => {
  // 在服务器端，可以通过 req,body 来获取 JSON 格式的表单数据和 url-encoded 格式的数据
  console.log(req.body)
  res.send('ok')
})
// 调用 app.listen 方法，指定端口号并启动web服务器
app.listen(80, function () {
  console.log('服务器已启动')
})
```

##### 5、第三方的中间件

```
非 Express 官方内置的，而是由第三方开发出来的中间件，叫做第三方中间件。在项目中，大家可以按需下载并配置第三方中间件，从而提高项目的开发效率。
例如：在 express@4.16.0 之前的版本中，经常使用 body-parser 这个第三方中间件，来解析请求体数据。使用步骤如下：
运行 npm i body-parser 安装中间件
使用 require 导入中间件
调用 app.use() 注册并使用中间件
注意：Express 内置的 express.urlencoded 中间件，就是基于 body-parser 这个第三方中间件进一步封装出来的。
```

body-parser中间件的使用

```js
// 导入 express 模块
const express = require('express')
// 创建 express 的服务器实例
const app = express()

// 1. 导入解析表单数据的中间件 body-parser
const parser = require('body-parser')
// 2. 使用 app.use() 注册中间件
app.use(parser.urlencoded({ extended: false }))
// app.use(express.urlencoded({ extended: false }))

app.post('/user', (req, res) => {
  // 如果没有配置任何解析表单数据的中间件，则 req.body 默认等于 undefined
  console.log(req.body)
  res.send('ok')
})

// 调用 app.listen 方法，指定端口号并启动web服务器
app.listen(80, function () {
  console.log('Express server running at http://127.0.0.1')
})
```



16.apiRouter

```js
const express = require('express')
const router = express.Router()

// 在这里挂载对应的路由
router.get('/get', (req, res) => {
  // 通过 req.query 获取客户端通过查询字符串，发送到服务器的数据
  const query = req.query
  // 调用 res.send() 方法，向客户端响应处理的结果
  res.send({
    status: 0, // 0 表示处理成功，1 表示处理失败
    msg: 'GET 请求成功！', // 状态的描述
    data: query, // 需要响应给客户端的数据
  })
})

// 定义 POST 接口
router.post('/post', (req, res) => {
  // 通过 req.body 获取请求体中包含的 url-encoded 格式的数据
  const body = req.body
  // 调用 res.send() 方法，向客户端响应结果
  res.send({
    status: 0,
    msg: 'POST 请求成功！',
    data: body,
  })
})

// 定义 DELETE 接口
router.delete('/delete', (req, res) => {
  res.send({
    status: 0,
    msg: 'DELETE请求成功',
  })
})

module.exports = router
```

17.测试接口跨域问题

```js
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>Document</title>
    <script src="https://cdn.staticfile.org/jquery/3.4.1/jquery.min.js"></script>
  </head>
  <body>
    <button id="btnGET">GET</button>
    <button id="btnPOST">POST</button>
    <button id="btnDelete">DELETE</button>
    <button id="btnJSONP">JSONP</button>

    <script>
      $(function () {
        // 1. 测试GET接口
        $('#btnGET').on('click', function () {
          $.ajax({
            type: 'GET',
            url: 'http://127.0.0.1/api/get',
            data: { name: 'zs', age: 20 },
            success: function (res) {
              console.log(res)
            },
          })
        })
        // 2. 测试POST接口
        $('#btnPOST').on('click', function () {
          $.ajax({
            type: 'POST',
            url: 'http://127.0.0.1/api/post',
            data: { bookname: '水浒传', author: '施耐庵' },
            success: function (res) {
              console.log(res)
            },
          })
        })

        // 3. 为删除按钮绑定点击事件处理函数
        $('#btnDelete').on('click', function () {
          $.ajax({
            type: 'DELETE',
            url: 'http://127.0.0.1/api/delete',
            success: function (res) {
              console.log(res)
            },
          })
        })
        // 4. 为 JSONP 按钮绑定点击事件处理函数
        $('#btnJSONP').on('click', function () {
          $.ajax({
            type: 'GET',
            url: 'http://127.0.0.1/api/jsonp',
            dataType: 'jsonp',
            success: function (res) {
              console.log(res)
            }
          })
        })
      })
    </script>
  </body>
</html>
```





## 八、获取请求体数据

#### 1、监听 req的data事件

```
在中间件中，需要监听 req 对象的 data 事件，来获取客户端发送到服务器的数据。
如果数据量比较大，无法一次性发送完毕，则客户端会把数据切割后，分批发送到服务器。所以 data 事件可能会触发多次，每一次触发 data 事件时，获取到数据只是完整数据的一部分，需要手动对接收到的数据进行拼接。
```

```js
//定义变量，用来处理客户端发送过来的请求体函数
let str = ''
//监听 req 对象的 data 时间（客户端发送过来的新的请求体数据）
req.on('data',(chunk)=>{
  //拼接请求体数据，隐式转换为字符串
  str += chunk
})
```



#### 2、监听req 的 end 事件

```
当请求体数据接收完毕之后，会自动触发 req 的 end 事件。
因此，我们可以在 req 的 end 事件中，拿到并处理完整的请求体数据。示例代码如下：
```

```js
//监听 req 对象的 end 事件（请求体发送完毕后自动触发）
req.on('end',()=>{
  //打印完整的请求体数据
  console.log(str)
  //TODO:把字符串格式的请求体数据，解析成对象格式
})
```



#### 3、使用 querystring 模块解析请求体数据

```
Node.js 内置了一个 querystring 模块，专门用来处理查询字符串。通过这个模块提供的 parse() 函数，可以轻松把查询字符串，解析成对象的格式。示例代码如下：
```

```js
//导入处理querystring的Node.js内置模块
const qs = require('querystring')

//调用 parse() 方法，把查询字符串解析为对象
const body = qs.parse(str)
```



#### 4、将解析出来的数据对象挂载为 req.body

```
上游的中间件和下游的中间件及路由之间，共享同一份 req 和 res。因此，我们可以将解析出来的数据，挂载为 req 的自定义属性，命名为 req.body，供下游使用。示例代码如下：

```

```js
req.on('end',()=>{
  const body = qs.parse(str) //调用 parse() 方法，把查询字符串解析为对象
  req.body = body //将解析出来的请求体对象，挂载为 req.body 属性
  next() //最后，一定要调用 next() 函数，执行后续的业务逻辑
})
```



![](4、express和中间件.assets/32.png)

```
注意：如果要获取 URL-encoded 格式的请求体数据，必须配置中间件 app.use(express.urlencoded({ extended: false }))
```

## 九、实例

#### 1、index.js

```js
let express = require("express")
let app = express();
let router = require("./router")
    //获取post传过来的数据必注册urlencoded,放到注册路由的前面
app.use(router)

app.listen("80", function() {
    console.log("服务器已启动");
})
```

#### 2、router.js//路由模板

```js
let express = require("express")
let router = express.Router();
router.get("/", function(req, res) {
    //console.log(typeof req.query);
    //用来接收get传过来的数据
    //let data = req.query;
    res.send({ status: 0, msg: "获取成功", data: req.query })
})
router.post("/reg", function(req, res) {
    //用来接收post传过来的数据
    console.log(req.body);
    res.send({
        status: 0,
        msg: "获取数据成功",
        data: req.body
    })
})
module.exports = router
```

#### 3、 使用express写jsonp接口

```js
// 导入 express
const express = require('express')
// 创建服务器实例
const app = express()
// 配置解析表单数据的中间件
app.use(express.urlencoded({ extended: false }))
// 必须在配置 cors 中间件之前，配置 JSONP 的接口
app.get('/api/jsonp', (req, res) => {
  //  定义 JSONP 接口具体的实现过程
  // 1. 得到函数的名称
  const funcName = req.query.callback
  // 2. 定义要发送到客户端的数据对象
  const data = { name: 'zs', age: 22 }
  // 3. 拼接出一个函数的调用
  //aa("{ name: 'zs', age: 22 }")
  const scriptStr = `${funcName}(${JSON.stringify(data)})`
  // 4. 把拼接的字符串，响应给客户端
  res.send(scriptStr)
})
// 一定要在路由之前，配置 cors 这个中间件，从而解决接口跨域的问题
const cors = require('cors')
app.use(cors())
// 导入路由模块
const router = require('./16.apiRouter')
// 把路由模块，注册到 app 上
app.use('/api', router)

// 启动服务器
app.listen(80, () => {
  console.log('express server running at http://127.0.0.1')
})
```



回顾JSONP 的概念与特点

```
概念：浏览器端通过 <script> 标签的 src 属性，请求服务器上的数据，同时，服务器返回一个函数的调用。这种请求数据的方式叫做 JSONP。
特点：
JSONP 不属于真正的 Ajax 请求，因为它没有使用 XMLHttpRequest 这个对象。
JSONP 仅支持 GET 请求，不支持 POST、PUT、DELETE 等请求。
```

创建 JSONP接口的注意事项

```
如果项目中已经配置了 CORS 跨域资源共享，为了防止冲突，必须在配置 CORS 中间件之前声明 JSONP 的接口。否则 JSONP 接口会被处理成开启了 CORS 的接口。示例代码如下：
```

![](4、express和中间件.assets/41.png)

实现 JSONP接口的步骤

```
获取客户端发送过来的回调函数的名字
得到要通过 JSONP 形式发送给客户端的数据
根据前两步得到的数据，拼接出一个函数调用的字符串
把上一步拼接得到的字符串，响应给客户端的 <script> 标签进行解析执行
```

. 实现JSONP接口的具体代码

![](4、express和中间件.assets/42.png)

在网页中使用 jQuery发起JSONP请求

```
调用 $.ajax() 函数，提供 JSONP 的配置选项，从而发起 JSONP 请求，示例代码如下： 
```

![](4、express和中间件.assets/43.png)



template模板

**模板引擎的基本概念**

**什么是模板引擎**

模板引擎，顾名思义，它可以根据程序员指定的模板结构和数据，自动生成一个完整的HTML页面。

![](E:\中软最新笔记\07node\整理node笔记\day03express\imgs\44.png)

模板引擎的好处

①减少了字符串的拼接操作

②使代码结构更清晰

③使代码更易于阅读与维护

**1.** **什么是标准语法**

art-template 提供了 **{{ }}** 这种语法格式，在 {{ }} 内可以进行**变量输出**，或**循环数组**等操作，这种 {{ }} 语法在 art-template 中被称为标准语法。

**标准语法** **-** **输出**

```js
{{value}}
{{obj.key}}
{{obj['key']}}
{{a ? b : c}}
{{a || b}}
{{a + b}}
```

在 {{ }} 语法中，可以进行变量的输出、对象属性的输出、三元表达式输出、逻辑或输出、加减乘除等表达式输出。

**标准语法** **–** **原文输出**

```
{{@ value }}
```

如果要输出的 value 值中，包含了 HTML 标签结构，则需要使用**原文输出**语法，才能保证 HTML 标签被正常渲染。 

**标准语法** **–** **条件输出**

如果要实现条件输出，则可以在 {{ }} 中使用 **if** … **else if** … **/if** 的方式，进行按需输出。

```
{{if value}} 按需输出的内容 {{/if}}
{{if v1}} 按需输出的内容 {{else if v2}} 按需输出的内容 {{/if}}
```

**标准语法** **–** **循环输出**

如果要实现循环输出，则可以在 {{ }} 内，通过 each 语法循环数组，当前循环的索引使用 **$index** 进行访问，当前的循环项使用 **$value** 进行访问。

```
{{each arr}}
    {{$index}} {{$value}}
{{/each}}
```

**标准语法** **–** **过滤器**

```
{{value | filterName}}
```

过滤器语法类似**管道操作符**，它的上一个输出作为下一个输入。

定义过滤器的基本语法如下：

```
template.defaults.imports.filterName = function(value){/*return处理的结果*/}
```

**标准语法** **–** **过滤器**

```
<div>注册时间：{{regTime | dateFormat}}</div>
```

定义一个格式化时间的过滤器 dateFormat 如下

```
template.defaults.imports.dateFormat = function(date) {
    var y = date.getFullYear()
    var m = date.getMonth() + 1
    var d = date.getDate()
    return y + '-' + m + '-' + d // 注意，过滤器最后一定要 return 一个值
 }
```



```js
  <!-- 2-准备模版 -->
    <script type="text/html" id="tmp">
        <h1>{{ title }}</h1>
        <ul>
            {{ each nav v i }}
                <li>{{ v }}</li>
            {{ /each }}
        </ul>   
    </script>

    <script src="../template-web.js"></script>
    <script>
        //后台服务器返回页面数据，需要渲染
        var obj = {
            title:'中软国际',
            nav: ['大前端', 'java', '大数据', 'python', 'UI设计', '产品经理', '新媒体']
        }    

        //渲染： 拼字符串
        // var str = '<h1>'+ obj.title +'</h1>';
        // str+='<ul>';
        //     obj.nav.forEach(function (v, i) {
        //        str += '<li>'+ v +'</li>';  
        //     })
        // str+='</ul>';

        // //添加页面
        // document.body.innerHTML = str;

        //在实际工作，拼接字符串是不可取， 之前数据量小，可以进行拼接，如果在实际工作中，正页面都是动态渲染的，如果还拼字符串， 容易出错，不容易排错,  项目维护非常麻烦！；

        //实际工作用 模版引擎， 
        // 模版引擎作用：渲染后台返回的数据

        //模版引擎使用步骤
        // 1- 引入插件
        // 2- 准备模版
        // 3- 准备数据
        // 4- 绑定数据和模版，生成结构
        // template(模版id, 数据);
       var str =  template('tmp', obj);

       console.log(str);

       document.body.innerHTML = str;

       //注意： 
       // 1- 模版接收的数据必须是对象{}，如果不是 要把数包装成对象，在进行转换
       // 2-在模版中直接使用属性名，不需要写对象名
```

