# egg

## 安装
直接使用脚手架，可快速生成项目文件夹

```js
npm i egg-init -g
egg-init egg-example --type=simple
cd egg-example
npm i
```

## 控制器
第一步需要编写的`Controller`和`Router`

```js
// app/controller/home.js（编写文件的位置）
const Controller = require('egg').Controller;
class HomeController extends Controller {
  async index() {
    this.ctx.body = 'Hello world';
  }
}
module.exports = HomeController;
```

配置路由映射

```js
// app/router.js（编写文件的位置）
module.exports = app => {
  const { router, controller } = app;
  router.get('/', controller.home.index);
};
```

## 静态资源
Egg 内置了 static 插件，线上环境建议部署到 CDN，无需该插件
static 插件默认映射`/public/* -> app/public/*`目录
此处，我们把静态资源都放到`app/public`目录即可

## 跨域
在`config/config.default.js`添加以下代码

```js
config.security = {
  csrf: {
    enable: false,
    ignoreJSON: true, // 默认为 false，当设置为 true 时，将会放过所有 content-type 为 `application/json` 的请求
  },
  domainWhiteList: [ 'http://localhost:8000' ],
};
config.cors = {
  allowMethods: 'GET,HEAD,PUT,POST,DELETE,PATCH,OPTIONS',
};
```

并且在`plugin.js`添加以下代码

```js
exports.cors = {
  enable: true,
  package: 'egg-cors',
};
```

## 服务
简单来说，Service 就是在复杂业务场景下用于做业务逻辑封装的一个抽象层，提供这个抽象有以下几个好处：

* 保持 Controller 中的逻辑更加简洁。
* 保持业务逻辑的独立性，抽象出来的 Service 可以被多个 Controller 重复调用。
* 将逻辑和展现分离，更容易编写测试用例，测试用例的编写具体可以查看[这里](https://eggjs.org/zh-cn/core/unittest.html)。

所以我们可以把操作数据库的逻辑放在 Service 层

定义 Service 文件

```js
// app/service/user.js
const Service = require('egg').Service;
class UserService extends Service {
  // 默认不需要提供构造函数。
  // constructor(ctx) {
  //   super(ctx); 如果需要在构造函数做一些处理，一定要有这句话，才能保证后面 `this.ctx`的使用。
  //   // 就可以直接通过 this.ctx 获取 ctx 了
  //   // 还可以直接通过 this.app 获取 app 了
  // }
  async find(uid) {
    // 假如 我们拿到用户 id 从数据库获取用户详细信息
    const user = await this.ctx.db.query('select * from user where uid = ?', uid);
    // 假定这里还有一些复杂的计算，然后返回需要的信息。
    const picture = await this.getPicture(uid);
    return {
      name: user.user_name,
      age: user.age,
      picture,
    };
  }
  async getPicture(uid) {
    const result = await this.ctx.curl(`http://photoserver/uid=${uid}`, { dataType: 'json' });
    return result.data;
  }
}
module.exports = UserService;
```

我们就可以在 Controller 层用`this.ctx.service.服务名xxx.方法xxx`来调用服务里面封装好的方法

```js
// app/router.js
module.exports = app => {
  app.router.get('/user/:id', app.controller.user.info);
};
// app/controller/user.js
const Controller = require('egg').Controller;
class UserController extends Controller {
  async info() {
    const { ctx } = this;
    const userId = ctx.params.id;
    const userInfo = await ctx.service.user.find(userId);
    ctx.body = userInfo;
  }
}
module.exports = UserController;
```

## 服务器代理
可以使用`curl`来代替第三方`request`模块，或者内置的`http.request`模块来实现服务器代理通讯

```js
class HomeController extends Controller {
  async news() {
    // 今日头条
    const { ctx } = this;
    const {
      data,
    } = await ctx.curl('https://m.toutiao.com/list/?tag=video&ac=wap&count=20&format=json_raw&as=A1457C764A41F74&cp=5C6AC19F07943E1&min_behot_time=0&_signature=1Y7F0AAAieymeM-.Mi2uANWOxc&i=', {
      dataType: 'json',
    });
    ctx.body = data;
  }
}
```
