# 海螺聊天室开发问题以及解决方案

开发pc端还是手机端，如果是多端要怎么适配

- 单独制作移动端和pc端页面
  
- 响应式布局
  
  根据屏幕尺寸来判断是pc端还是移动端 进而改变页面的渲染代码
  
- 自适应布局
  
  只是对页面的尺寸进行缩放 没有改变页面布局 严格来说不算解决方案 移动端一般不需要 因为视口的宽度固定 电脑需要
  

方案：自适应过时对于跨端来说，响应式过于复杂且提升代码复杂度 使用双端界面方案

js的浏览器api判断是否是手机 用vue的路由跳转完成多端

```
string下的match方法 和 RegExp.prototype.test() 都是正则表达的布尔匹配

if(/Android|webOS|iPhone|iPod|BlackBerry/i.test(navigator.userAgent)) {
    window.location.href = "wap";
} else {
    window.location.href = "pc";
}
```

## 22-7-3

- ### 完成数据库创建和尝试用koa脚手架创建后端项目 对架手架给的koa中间件进行删改
  
  ```
      "debug": "^4.1.1",
      "koa": "^2.7.0",
      "koa-body": "^5.0.0", 将post的body直接放到ctx.body中 简化node原生事件获取请求体代码 同时需要文件上传 而 koa-bodyparser无法处理文件 所以不使用koa-bodyparser 和 koa-multer
      "koa-json": "^2.0.2",格式化json格式 暂时不用
      "koa-logger": "^3.2.0",
      "koa-onerror": "^4.1.0",
      "koa-router": "^7.4.0",
      "koa-static": "^5.0.0",获取静态资源文件
      "mysql": "^2.18.1",数据库操作
      "pug": "^2.0.3"
      "jsonwebtoken": "^8.5.1", 生成和校验jwt
  
      删除了
      koa-view 因为项目为前后端分离 koa-view为后端提供html文件 不需要
      koa-bodyparser
  ```
  
- ### 创建接口文档规范
  
  http轮询原本是要分多个接口 后面决定只走一个接口其中需要获得的数据包含<群新消息 私聊新消息 群被踢出 私聊被删 账号被冻结 有新好友
  

## 22-7-4

描述：写接口文档后一些技术的实现方案的探索

目标：对项目模块进行分析 对功能模块进行完善

- 业务本身 需要springboot的三层级划分 目录完善
- 图片上传koa-body和图片获取koa-static系统
- 账号登录登出 延长cookie有效时间 jwt校验 获取cookie信息 （jsonwebtoken）
- 全局环境（上下文）的完善（写入用户信息）

其他操作

- db连接池取消放在koa的ctx下
  
- ### 文件上传解决方案
  
  前端部分
  
  ```
  document.querySelector("input").addEventListener('input',(e)=>{
              const file =  e.target.files[0]
              let param = new FormData()  // 创建form对象
              param.append('file', file, 'name')  // 通过append向form对象添加数据
              param.append('lin',"hello")
              let config = {
              headers: {}
          }
  
              axios({
                  url: 'http://127.0.0.1:3000/string',
                  method: 'post',
                  headers:{
                      'Content-Type': 'multipart/form-data'
                  },
                  data:param
              }).then(response => {
  
              console.log(response)
              })
  
          })
  1.使用axios 在input选择完图片触发事件
  2.使用事件下的方法const file =  e.target.files[0]获取文件数据
  3.创建formdata表单  let param = new FormData() 
  4.文件写入formdata
  5.axios发送文件
  ```
  
  后端部分
  
  ```
  使用axios-body中间件
  
  1.引入中间件multipart: true表示支持文件
  formidable是对文件上传的其他参数设置
  uploadDir为文件的保存位置
  keepExtensions为是否保存文件后缀名
  app.use(koaBody({ multipart: true})，
  formidable:{
      uploadDir:"C:\\Users\\TIGO\\Desktop",
      keepExtensions:true
    })
  ```
  
- koa跨域配置
  
  ```
  app.use(async (ctx, next)=> {
    ctx.set('Access-Control-Allow-Origin', '*');
    ctx.set('Access-Control-Allow-Headers', 'Content-Type, Content-Length, Authorization, Accept, X-Requested-With , yourHeaderFeild');
    ctx.set('Access-Control-Allow-Methods', 'PUT, POST, GET, DELETE, OPTIONS');
    if (ctx.method == 'OPTIONS') {
      ctx.body = 200; 
    } else {
      await next();
    }
  });
  ```
  

## 22-7-5

描述：完成昨天剩下的清单，编写请求参数校验中间件parameter.js

- 自写简单参数校验 写两个方法 分别对url传参和请求头传参校验
  
  对于url传参 koa解析后的对象只会是string类型 所以需要在参数校验时候转换类型
  
- 对koa的ctx.body不进行设置koa将视为404
  
- isNaN()和Number.isNaN()区别
  
  isNaN()尝试转换为数字转换成功为false 转化失败为true ，Number.isNaN()只有当NaN时候才为true
  
  在项目中 我用isNaN()来判断字符串是否为纯数字
  

## 22-7-6

描述：

1. 尝试打通一个完整接口 进行开发规范制定
  
2. mysql数据库操作是异步的 所以将router service mapper 三层中的方法都改为async调用
  
3. router把koa的ctx传给service，由service写入ctx.body
  
4. mapper对数据库操作后 返回对象表达成功 返回undifined表示失败
  

- mysql包小坑
  
  ```
   `insert into user(profile_picture,nickname,pass,age,sex,autograph) values('${fileName}','${username}','${password}',${age},${sex},'${autograph}')`
  ```
  
  mysql包不会为字符串sql语句中的字符串自动填充 ' ' 需要手动添加
  

## 22-7-7

描述：开发接口

- mysql的使用中 在官方有两种方式
  
  1
  
  connection.query(`SELECT * FROM `books` WHERE `author`= "${David}"`, function (error, results, fields)
  
  2
  
  connection.query('SELECT * FROM `books` WHERE `author` = ?', ['David'], function (error, results, fields)
  
  第一种使用了模板字符串 第二种 sql和参数分开传参
  
  后面使用第二种 因为第一种方式当sql语句需要转义的时候会和模板字符串冲突
  

## 22-7-8

描述：开发接口

- mysql
  
  timestamp转时间戳 函数unix_timestamp()
  
  时间戳转timetamp 函数FROM_UNIXTIME()
  
- js时间戳
  
  js时间戳精确到毫秒数 有13位 而mysql的时间戳单位是秒 有10位
  
  [Unix时间戳(Unix timestamp)转换工具 - 站长工具 (chinaz.com)](https://tool.chinaz.com/Tools/unixtime.aspx)
  

## 22-7-9

描述：开发接口

剩下三分二的群聊模块和http轮询 预计一天完成 最多两天完成 如何就可以开始前端开发

- 陌生人匹配模块的部分接口的service复用用户模块
  
- js取随机整数 包含最大最小值
  
  ```js
  function getRandomIntInclusive(min, max) {
    min = Math.ceil(min);
    max = Math.floor(max);
    return Math.floor(Math.random() * (max - min + 1)) + min; //含最大值，含最小值 
  }
  ```
  
- 需要对时间字符串表达格式化
  
  - 简单的Date时间 格式化 （固定的格式）（ 自定义格式方法暂不写）
  
  ```js
  `${now.getFullYear()}-${now.getMonth()}-${now.getDate()} ${now.getHours()}:${now.getMinutes()}:${now.getSeconds()}`
  ```
  
  - 后面发现date有专门方法 toLocaleString()

### 完善和理清 举报用户冻结机制

细见文档

# bug

- 前端上传文件使用formdata对象时 param.append('file', file, 'name') 第三个参数为文件名称 默认为原始名称 如果重写时候不屑文件后缀后端将获取不到文件后缀名

# 遗留

- ### 内容：stream is not readable
  
  koa-body和koa-router结合使用中有这么一段
  
  [koa-body - npm (npmjs.com)](https://www.npmjs.com/package/koa-body) 官网
  

## Usage with [koa-router](https://github.com/alexmingoia/koa-router)

It's generally better to only parse the body as needed, if using a router that supports middleware composition, we can inject it only for certain routes.（router.post方法中加入koaBody()是可选的）

```
const Koa = require('koa');
const app = new Koa();
const router = require('koa-router')();
const koaBody = require('koa-body');

router.post('/users', koaBody(),
  (ctx) => {
    console.log(ctx.request.body);
    // => POST body
    ctx.body = JSON.stringify(ctx.request.body);
  }
);

app.use(router.routes());

app.listen(3000);
console.log('curl -i http://localhost:3000/users -d "name=test"');
```

koaBody()加入后body会报错 去掉后正常

- ### isNaN 与空字符串
  
  isNaN的目的是一个值是否可以为NaN，其中这个函数会尝试将参数转换为数字 一般的非数字表达会返回true
  
  可以转为数字返回false 其中空字符串将得到false 但是使用parseInt("")却会得到NaN
  
  场景：node后端校验时候使用了isNaN 然后发现无法过滤空字符串
