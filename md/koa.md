# koa配置

[基础设置](https://www.jianshu.com/p/4a458e14cb73)

### 简单CORS设置

```js
//app.js

//需安装koa2-cors
const cors = require('koa2-cors');

app.use(cors({
    origin: function (ctx) {
        if (ctx.url === '/test') {
            return "*"; // 如果等于/test，就允许来自所有域名请求
        }
        return 'http://localhost:8080'; / 这样就能只允许 http://localhost:8080 这个域名的请求了
    },
    exposeHeaders: ['WWW-Authenticate', 'Server-Authorization'],
    maxAge: 5,
    credentials: true,
    allowMethods: ['GET', 'POST', 'DELETE'],
    allowHeaders: ['Content-Type', 'Authorization', 'Accept'],
}))

```
