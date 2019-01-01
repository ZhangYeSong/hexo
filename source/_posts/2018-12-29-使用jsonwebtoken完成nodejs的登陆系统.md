---
title: 使用jsonwebtoken完成nodejs的登陆系统
date: 2018-12-29 22:02:04
categories:
 - Node.js
tags:
 - Node.js
 - Koa
---
今天我们继续，做一个简单的登陆系统，使用[jsonwebtoken](https://jwt.io/)作鉴权。

### Mongoose添加数据库账号集合
首先我们定义一个AccountSchema，如下：
```
const mongoose = require('mongoose');
const Schema = mongoose.Schema;

const AccountSchema = new Schema({
    user_id: {type: String, required: true},
    username: {type: String, required: true},
    password: {type: String, required: true},
});

const AccountModel = mongoose.model('Account', AccountSchema);
module.exports = AccountModel;
```
然后我们创建一个名为account的路由，开始写接口。先写注册接口，
注册接口就是直接post account，代码如下：
<!-- more -->
```
const response = require('../util/response-util');
const router = require("koa-router")();
const AccountModel = require('../model/account');
const key = require('../config/secret-key');
const md5 = require('md5');

//注册账号接口
router.post('/', async (ctx) => {
    let requestAccount = ctx.request.body;
    if (!requestAccount.username || requestAccount.username.length < 3) {
        ctx.throw(400, 'length of username need >= 3');
    }
    if (!requestAccount.password || requestAccount.password.length < 6) {
        ctx.throw(400, 'length of password need >= 6');
    }

    const isDuplicatedUsername = await AccountModel.findOne({'username': requestAccount.username});
    if(isDuplicatedUsername) {
        ctx.throw(400,'duplicated username');
    }

    requestAccount.password = md5(key.accountPasswordKey + requestAccount.password);
    let result = await AccountModel.create(requestAccount);
    if (result) {
        ctx.body = response.createOKResponse(result);
    } else {
        ctx.body = response.createFailedResponse(500, 'create account failed');
    }

});
```
账号使用password再加一个盐指一起做MD5加密，虽然对Schema里做了长度、唯一性限制，在接口上也要做限制来做json返回。
再写一个注销接口方便写做单元测试：
```
//注销账号接口
router.delete('/', async (ctx) => {
    let username = ctx.query.username;
    if (!username) {
        ctx.throw(400, 'need username');
    }

    let result = await  AccountModel.findOneAndDelete(username);
    if (result) ctx.body = response.createOKResponse(result);
    else ctx.body = response.createFailedResponse(500, 'delete account fail')
});
```

然后是登陆接口，先做校验，生成token令牌下面再做：
```
//登陆接口
router.post('/user-token', async (ctx) => {
    let requestAccount = ctx.request.body;
    if (!requestAccount.username || requestAccount.username.length < 3) {
        ctx.throw(400, 'length of username need >= 3');
    }
    if (!requestAccount.password || requestAccount.password.length < 6) {
        ctx.throw(400, 'length of password need >= 6');
    }

    requestAccount.password = md5(key.accountPasswordKey + requestAccount.password);
    let account = await AccountModel.findOne(requestAccount);
    if (account) {
        ctx.body = response.createOKResponse(account);
    } else {
        ctx.body = response.createFailedResponse(500, 'query account failed');
    }

});
```
这些都很简单。退出登陆不需要做，根据jwt的标准，后端不保存token，想要退出登陆前端把当前token删掉就可以了。

### 使用JWT作登陆认证
现在的应用都要作多终端认证，因此使用token来做认证是非常合适的，基本上移动端都是用accsess-token来验证用户的登陆信息。
[jsonwebtoken](https://jwt.io/)是一个跨域认证标准，不了解的朋友可以先看看阮一峰老师的这篇博客[JSON Web Token 入门教程](http://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html)。
它的好处就是可以跨域，跨平台。而且由于服务端不需要保存token信息，开发起来非常简单。
JWT在不同语言、平台有不同的实现库，由于我们是nodejs，所以直接去npm上找，[node版JWT](https://www.npmjs.com/package/jsonwebtoken)这个就是了，可以看下使用方法，真的是非常简单呢。先运行下npm install jsonwebtoken安装JWT依赖。
下面我们继续写登陆接口，返回一个user-token给客户端，客户端拿到后保存起来，以后的请求在请求头里加上token信息，后端就可以识别了。
由于我打算用加密强度非常大的RSA256加密jwt，因此先生成一对RSA密钥，我们借助openssl来创建RSA256密钥对：
```
full-stacker/full-stacker-api/config  master ✗                                                                                                                                           2d ⚑  
▶ openssl
OpenSSL> genrsa -out jwt.pem 1024                        
Generating RSA private key, 1024 bit long modulus
....++++++
.......................++++++
e is 65537 (0x10001)
      
OpenSSL> rsa -in jwt.pem -pubout -out jwt_pub.pem
writing RSA key
OpenSSL> exit

full-stacker/full-stacker-api/config  master ✗                                                                                                                                         2d ⚑ ◒  
▶ ls
jwt.pem       jwt_pub.pem   mongo-db.js   secret-key.js
```
密钥有了，我们可以在登陆接口生成jwt给客户端了：
```
//登陆接口
router.post('/user-token', async (ctx) => {
    let requestAccount = ctx.request.body;
    if (!requestAccount.username || requestAccount.username.length < 3) {
        ctx.throw(400, 'length of username need >= 3');
    }
    if (!requestAccount.password || requestAccount.password.length < 6) {
        ctx.throw(400, 'length of password need >= 6');
    }

    requestAccount.password = md5(key.accountPasswordKey + requestAccount.password);
    let account = await AccountModel.findOne(requestAccount);
    if (account) {
        let cert = fs.readFileSync(path.resolve(__dirname, '../config/jwt.pem'));
        let userToken = jwt.sign({
                _id: account._id,
                username: account.username
            }, cert,
            {
                algorithm: 'RS256',
                expiresIn: '1h'
            });
        ctx.body = response.createOKResponse(userToken);
    } else {
        ctx.body = response.createFailedResponse(500, 'wrong username or password');
    }

});
```
这其中，fs读取文件的时候读相对路径经常会找不到，无奈借助path来读绝对路径给fs，jwt里存储用户的id和username，这里expiresIn表示过期时间。
最后我们写个测试接口来测试一个jwt登陆鉴权：
```
//登陆鉴权测试接口
router.get('/test', async (ctx) => {
    userToken = ctx.request.get('Authorization');
    let cert = fs.readFileSync(path.resolve(__dirname, '../config/jwt_pub.pem'));

    try {
        const decoded = await jwt.verify(userToken, cert);
        ctx.body = response.createOKResponse(decoded);
    } catch (e) {
        ctx.throw(401, 'need authorization')
    }
});
```
![Restlet Client](https://upload-images.jianshu.io/upload_images/5586297-b2f04651b045ded3.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
这里推荐一个测试接口的chrome插件[Restlet Client](https://chrome.google.com/webstore/detail/restlet-client-rest-api-t/aejoelaoggembcahagimdiliamlcdmfm)，感觉比postman更好用。

现在我们再把鉴权过程封装成方法，新建一个util文件login-authorization.js，如下：
```
const jwt = require('jsonwebtoken');
const fs = require('fs');
const path = require('path');

module.exports = async function (ctx) {
    const requestToken = ctx.request.get('Authorization');
    let cert = fs.readFileSync(path.resolve(__dirname, '../config/jwt_pub.pem'));
    try {
        return await jwt.verify(requestToken, cert);
    } catch (e) {
        ctx.throw(401);
    }
};

这样其他地方要获取解析后的jwt直接调用该方法就可以了。
```
好了，这下我们基本的账号系统和登陆鉴权就做完了，使用JWT是不是超级简单呢？

### 编写单元测试
使用ava+superkoa做单元测试，代码如下：
```
import test from 'ava';
import superKoa from 'superkoa';
import app from '../app';

test.serial('register account', async t => {
    let res = await superKoa(app)
        .post('/account')
        .send({username: 'test-account', password: '123456'});
    t.is(200, res.status);
    t.is(0, res.body.error);
    t.is('test-account', res.body.data.username);
});

test.serial('login && authorization', async t => {
    let res = await superKoa(app)
        .post('/account/user-token')
        .send({username: 'test-account', password: '123456'});
    t.is(200, res.status);
    t.is(0, res.body.error);

    let res2 = await superKoa(app)
        .get('/account/test')
        .set('Authorization', res.body.data);
    t.is(200, res2.status);
    t.is(0, res2.body.error);
    t.is('test-account', res2.body.data.username);
});

test.serial('unregister authorization', async t => {
    let res = await superKoa(app)
        .delete('/account?username=test-account');
    t.is(200, res.status);
    t.is(0, res.body.error);
    t.is('test-account', res.body.data.username);
});
```
运行一下，看看结果：
```
▶ npm test

> full-stacker-server@0.1.0 test /Users/judy/WeChatProjects/full-stacker/full-stacker-api
> ava -v

POST /category - 424ms
  --> POST /category 200 431ms 89b
  ✔ create category (473ms)
  <-- PATCH /category
PATCH /category - 912ms
  --> PATCH /category 200 913ms 90b
  ✔ update category (920ms)
  <-- GET /category/list?parent=root
GET /category/list?parent=root - 22ms
  --> GET /category/list?parent=root 200 31ms 270b
  ✔ query categories by parent
  <-- DELETE /category?_id=test
DELETE /category?_id=test - 21ms
  --> DELETE /category?_id=test 200 22ms 90b
  ✔ delete category
  <-- POST /account
POST /account - 56ms
  --> POST /account 200 58ms 133b
  ✔ register account
  <-- POST /account/user-token
POST /account/user-token - 25ms
  --> POST /account/user-token 200 25ms 356b
  <-- GET /account/test
GET /account/test - 6ms
  --> GET /account/test 200 8ms 113b
  ✔ login && authorization
  <-- DELETE /account?username=test-account
DELETE /account?username=test-account - 15ms
  --> DELETE /account?username=test-account 200 17ms 133b
  ✔ unregister authorization
  <-- GET /
GET / - 0ms
  --> GET / 200 3ms 19b
  ✔ hello full-stacker

  8 tests passed
```
全部通过（有5个是之前的，有时间把测试包给分一下），happy，回家过元旦！
