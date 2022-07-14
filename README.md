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
  
  ```js
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
  

### 完善和理清 举报用户冻结机制

细见文档

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

## 22-7-10

描述：完成剩下的接口开发

- mysql包中执行 update 语句时候 其返回的对象下的affectedRows属性是where匹配到的行数 而不是受到改变的行数 注意一下
  
- 群聊天的聊天记录获取，按理来说前端显示聊天记录的同时应该显示用户的头像等信息 者导致后端还要连表查询用户信息，但是有一个接口是查询群成员信息，所以这部分交给前端处理
  
- 群冻结和用户冻结的判定机制 一样 但是不需要考虑登录等情况，只需在用户发言时候：如果群冻结则阻止
  

### http轮询设计方案

##### 思路

对于后端来说是一个接口，需要考虑查询哪些数据，哪些数据需要更新

对于前端来说， 是多个请求分别对应页面各个内容，还是只有一个轮询请求完成对所有数据的更新。

显然，第一种会导致多个http轮询 不好管控，且会造成频繁的网络请求，效率 （作用/消耗的资源） 低下。第二种只使用一个轮询，好管控，但是前端要布局好数据的传递，http请求的数据所在页面/域 传递到另一个组件/页面 像一个树型结构

##### 方案

后端：

使用前端传递的时间戳和数据库的create/update（创建/更新时间字段）进行比较 大于当前时间视为更新 加入http数据之中

更新的字段:

1. 消息(群新消息,私聊新消息) 两个消息表中的 create字段
  
2. 被动删(群被踢出,私聊被删) 两个关系表中的 update字段
  
3. 被动新增(有新好友,新群友) 两个关系表中的 create字段
  

http轮询需要一个时间戳来**界定新旧数据** 而这个时间戳的获取来源有两种方案

1：由用户登录后 初始数据里记录的数据库时间戳(前端获取好友列表后 得到好友列表里的最后更新时间 交由轮序接口)

2：在后端接受到请求后生成当前时间 然后查询 把时间也返回给后端 作为下次更新时间戳

第一种的时间戳获取更加的**精确** 第二种**开销小** 但是可能产生重复数据，为什么呢 因为在后端的步骤是：

获取前端传来的上一次**更新时间戳**

=>获取**当前时间戳**

=> 使用**前端更新时间戳**执行查询数据库，加入js队列

=>回调，获取数据 返回前端，同时带上**当前时间戳**作为下一次的**更新时间戳**

但是js是单线程,存在一种情况：后端在获取 **当前时间戳** 到 查询完数据 期间 **有新的记录加入到了数据库** 这将导致数据库查询获取的数据的**时间点**存在大于**当前时间戳** 的数据 导致这类数据在下次轮询时也会被查询到 导致重复数据 所以 前端需要实现对重复数据的过滤

三个列表的更新（群列表 好友列表，匿名列表）的排重简单

但是消息的排重需要一定的考究 假如前端当前有十条记录 两次分页 一页五条 每次新聊天记录获取后我只会对当前数据最新五条来参考排重 而不是十条

同时消息的结构更加复杂 列表的更新都是同一个数组下 而消息的更新是对应不同对象的聊天记录数组

## 22-7-11

描述：开始构思前端 http轮询接口到前端具体使用时再写

1 使用的框架vue vue全家桶的使用 router/vuex, vue脚手架中的配置选择/插件 babel等

2 项目结构

3 工具包：防抖，节流方法的生成函数 可能需要 promise

4 axios 发送http请求的封装 配置 api编写形式

5 移动端还是pc端

6 页面样式 前端的布局 对应的数据结构 组件通信

- vue add 和npm install区别
  
  npm的引入是单纯的把目标资源下载
  
  而vue add在npm引入的基础上 提供一些由编写者预设的一些基础配置，它会更改当前文件结构
  
- vuecli初始化配置preset
  
  - Choose Vue version vue版本选择 （vue3）
    
  - Babel 是否兼容低版本浏览器 js编译器 把高级语法转为浏览器支持的语法
    
  - TypeScript 是否扩展JavaScript （不使用）
    
  - Progressive Web App (PWA) Support 渐进式Web应用程序(web服务基于浏览器 pwa是web服务一个较新的概念 目标：使web服务像一个应用程序一样使用 ) (不使用)
    
  - Router 是否配置路由 （使用）
    
  - Vuex 是否配置状态管理模式（相当于本地全局数据存储仓库）
    
  - CSS Pre-processors CSS预处理器（使用）
    
  - Linter / Formatter 格式化程序规范选择 （使用）
    
  - Unit Testing 是否创建单元测试
    
  - E2E Testing 是否创建端到端测试
    
- vue 中 linter 的4种配置及其官方说明
  
  1、[ESLint](https://so.csdn.net/so/search?q=ESLint&spm=1001.2101.3001.7020) with error prevention only
  
  > 只配置使用 ESLint 官网的推荐规则  
  > 这些规则在这里 [添加链接描述](https://eslint.bootcss.com/docs/rules/)
  
  2、ESLint + Airbnb config 爱彼迎规范
  
  > 使用 ESLint 官网推荐的规则 + Airbnb 第三方的配置  
  > Airbnb 的规则在这里 [添加链接描述](https://github.com/airbnb/javascript)
  
  3、ESLint + Standard config 通用规范
  
  > 使用 ESLint 官网推荐的规则 + Standard 第三方的配置  
  > Standard 的规则在这里 [添加链接描述](https://github.com/standard/standard/blob/master/docs/RULES-zhcn.md#javascript-standard-style)
  
  4、ESLint + Prettier 比较漂亮的规范
  
  > 使用 ESLint 官网推荐的规则 + Prettier 第三方的配置  
  > Prettier 主要是做风格统一。代码格式化工具[Options · Prettier](https://prettier.io/docs/en/options.html)
  
  这里我使用第四种
  

## 22-7-12

描述：搭建前端项目

- 防抖和节流(重点在思想 具体实现不同人的代码细节不同)
  
  防抖：一段时间内的频繁触发只执行最后一次
  
  ```js
  const antiShake = (func,delay)=>{
      let timeouter;
      return function(e){
          clearTimeout(timeouter);
          timeouter = setTimeout(()=>{
              func(e)
          }, delay); 
      }
  };
  ```
  
  节流：控制多次执行之间的时间间隔最小值
  
  节流有两种写法 一种是settimeout 一种是用时间戳 我选择时间戳 因为我把节流用于发起请求的场景 时间戳一旦触发就可以执行 但是settimeout有延迟 减低用户使用体验
  
  ```js
  const antiShake = (func,delay)=>{
      let timeouter;
      return function(e){
          clearTimeout(timeouter);
          timeouter = setTimeout(()=>{
              func(e)
          }, delay); 
      }
  };
  ```
  
  ps：我写的这两种函数默认为一个传参 方式 多个传参方式需要arguments
  

## 22-7-13

描述：开始尝试写前端登录页面（挺久没动vue全家桶，本来就不算熟练更是忘了）

- 登录页面：选用一张大规模的图片作为div背景 div的大小为全屏，因为不同屏幕的宽高比是不一样的，所以不能奢求一张图片和屏幕完全吻合,所以使用铺满不重复来应对不同屏幕
  
  ```css
      background-size: cover;
      background-repeat: no-repeat;
  ```
  
- css 的 :root 和var()
  
  在element plus的教程中的颜色配置中发现这一个新的知识点[Color 色彩 | Element Plus](https://element-plus.gitee.io/zh-CN/component/color.html)
  
  :root是一个伪类 其中的键值对可以在其他css代码中通过var(key得到)
  
  在:root中配置通用值以在其他css使用
  
- vue生命周期与js代码
  
  ```js
  <script>
  const file =  document.querySelector(".test")
  const a = 111111111
  export default {
    name:"LoginView",
    beforeCreate(){
  
    },
    mounted() {
      console.log(a);
      const clientHeight = document.documentElement.clientHeight;
      const img =  document.querySelector(".background");
      img.style.height = clientHeight + "px"
    },
    data(){
      return {
          input:"rdfrtg"
      }
    },
    methods:{
      chooseFile(){
  
          file.clic
  k()
      }
    }
  }
  </script>
  ```
  
  场景：在创建vue实例的代码中export default {} 我需要在外部声明变量并在实例中使用（const file = document.querySelector(".test")） 但实际上 我拿不到file对象 会是null ,html代码也没有错，我的理解是 在vue实例的生命周期中先读取script的代码生成实例，但此时并没有对html渲染，导致file无法获取对象，这段代码的赋值应当在mounted函数中
  
- **图片选择及其预览**
  
  原生的input文件选择不好看 而且要图片预览不只显示文件名称，所以把input的宽度设置为0，替代的按钮 点击时候触发input的click事件，
  
  图片选择后展示：
  
  ```js
      getFile(e){
              let reader = new FileReader();
              reader.onload = function () {
                  document.querySelector('.background-main-show').src = this.result;
              };
              // 设置以什么方式读取文件，这里以base64方式
              reader.readAsDataURL(e.target.files[0]);
          },
  ```
  
  FileReade为文件读取对象
  
  onload 回调函数 文件读取完成后调用
  
  src 修改图片元素的内容
  
  readAsDataURL 开始读取
  

## 22-7-14

描述：登录页面本身内容写完，打通一套完整的封装，项目结构，防抖节流 axios封装 等的完善（**实践获取经验，第一次真正意义上的自己写的vue项目，很多封装，简单轮子自己摸索**）

- 退出登录并不需要后端 删除此接口
  
- 起因：封装axios，实践后发现：在js对象转json中 undefined的属性将不会转化到json中，情理之中
  

# bug

- 前端上传文件使用formdata对象时 param.append('file', file, 'name') 第三个参数为文件名称 默认为原始名称 如果重写时候不屑文件后缀后端将获取不到文件后缀名

# 遗留/未知问题

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
