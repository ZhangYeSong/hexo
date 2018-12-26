---
title: 使用koa2+mongodb+ava构建RESTful api并测试
date: 2018-12-24 19:55:28
categories:
 - Node.js
tags:
 - Node.js 
 - MongoDB
 - mongoose
 - ava
---
> 初涉nodejs后台开发，在得知express和koa是同一个团队开发之后果断选择了更前沿的koa2试水。结果发现koa生态是真的不成熟啊，不过开发起来也更有意思。

### 安装依赖
此次我们使用koa-generator作为脚手架创建项目，这个也不是官方脚手架，大家熟悉了可以随便改。
```
npm install koa-generator -g
koa2 my-project
cd my-project
```
这样就生成了一个基本的项目框架。
除此之外，必须要安装的还有用来操作MongoDB的mongoose、用来进行单元测试的框架ava以及superkoa。
```
npm install --save mongoose
npm install --save-dev ava
npm install --save-dev superkoa
```
下面是我项目的package.json文件的依赖，其中mount-koa-routes是一个自动读取routes文件的框架，可以不用。
<!-- more -->
```
"dependencies": {
    "debug": "^2.6.3",
    "koa": "^2.2.0",
    "koa-bodyparser": "^3.2.0",
    "koa-convert": "^1.2.0",
    "koa-json": "^2.0.2",
    "koa-logger": "^2.0.1",
    "koa-onerror": "^1.2.1",
    "koa-router": "^7.1.1",
    "koa-static": "^3.0.0",
    "koa-views": "^5.2.1",
    "mongoose": "^5.4.0",
    "mongoosedao": "^1.0.13",
    "mount-koa-routes": "^2.0.1",
    "pug": "^2.0.0-rc.1"
  },
  "devDependencies": {
    "ava": "^1.0.1",
    "nodemon": "^1.18.9",
    "superkoa": "^1.0.3",
  }
```
### 定义数据库连接和Schema对象
此次我们要做的api是做一个能够增删改查分类的api。MongoDB是这几年非常火爆的一个nosql数据库。在尝试过后确实赶紧nosql很爽。MongoDB的学习资料建议看这个：http://www.runoob.com/mongodb/mongodb-tutorial.html
搭建一个MongoDB数据库要比Mysql和OracleDB快多了。
我们使用mongoose.js来处理MongoDB的相关操作。mongoose.js可以看作是nodejs上MongoDB的ORM框架，和java后端的hibernate以及android端的greenDAO类似。有过ORM框架经验的上手非常容易。具体的mongoose学习看官网就非常好：https://mongoosejs.com/
首先我们在新建一个mongo-db.js来执行MongoDB的连接：
```
const mongoose = require('mongoose');

const connect = mongoose.connect('mongodb://www.zhangyesong.com:27017/test',
    {useNewUrlParser: true});
connect.then((() => {
    console.log('连接数据库成功');
}), (error => {
    console.log('连接数据库失败' + error);
}));
```
是不是很简单？然后在app.js引用就可以：
```
require('./config/mongo-db');
```

新建一个目录用来存放所有的Scheme，在里面新建一个文件category.js，如下：
```
const mongoose = require('mongoose');
const Schema = mongoose.Schema;

const CategorySchema = new Schema({
    _id: String,
    name: {type: String, required: true},
    parent: String,
    level: {type: Number, min: 0, max: 5},
});

const CategoryModel = mongoose.model('Category', CategorySchema);
module.exports = CategoryModel;
```
虽然MongoDB没有表的概念只有Collection的概念，但是正常情况下我们肯定还是让Collection里的每条数据都有着相同的数据结构的。Schema就是定义Colletion里面数据的结构的。
这里我们给category定义了四个key。
_id是默认的数据的默认字段，全局唯一，默认类型是ObjectId，这里我改成了String，由用户自己来定义。
name记录分类的名字。
parent记录分类的上级分类。
level记录分类的层级。
定义好Schema之后，创建model类然后export出去。
### 撰写RESTful api
RESTful api设计风格越来越流行，接口不做成RESTful怎么行呢？
我所掌握的RESTful api有以下两个要点：
- url要使用表示资源的名字，比如我这里就是category或categories，尽量不要使用动词。
- 使用GET来做query请求，POST做创建请求，PATCH做修改请求，DELETE做删除请求，反正就是把Http请求用对，不用什么都用GET和POST一把干完。

想详细学习以下RESTful的同学可以看看阮一峰老师的这篇博客：http://www.ruanyifeng.com/blog/2018/10/restful-api-best-practices.html
这里面争议比较大的是query请求灵活多变，url上有时候强行用资源名称反而会导致接口语义不清。因此我这里只保证query以外的请求符合REST风格。

首先我们对所有的返回简单封装以下：
```
exports.createOKResponse = function(data) {
    return {
        error: 0,
        data: data,
    }
};

exports.createFailedResponse = function(error, message) {
    return {
        error: error,
        message: message,
    }
};
```

然后api接口代码如下：
```
const response = require('../util/response-util');
const router = require("koa-router")();
const CategoryModel = require('../model/category');

router.post('/', async (ctx) => {
    let requestCategory = ctx.request.body;
    if (!checkCategory(requestCategory)) {
        ctx.body = response.createFailedResponse(400, 'bad request params');
        return;
    }

    let result = await CategoryModel.create(requestCategory);
    if (result) {
        ctx.body = response.createOKResponse(result);
    } else {
        ctx.body = response.createFailedResponse(500, 'create category failed');
    }
});

router.delete('/', async (ctx) => {
    let _id = ctx.query._id;
    if (!_id) {
        ctx.body = response.createFailedResponse(400, 'bad request params');
        return;
    }

    let result = await CategoryModel.findByIdAndDelete({_id});
    if (result) ctx.body = response.createOKResponse(result);
    else ctx.body = response.createFailedResponse(500, 'delete category fail')
});

router.patch('/', async (ctx) => {
    let requestCategory = ctx.request.body;
    let _id = requestCategory._id;
    if (!_id) {
        ctx.body = response.createFailedResponse(400, 'bad request params');
        return;
    }

    let category = await CategoryModel.findById(_id);
    if (!category) {
        ctx.body = response.createFailedResponse(404, 'can not find such category');
        return
    }

    if(requestCategory.name) category.name = requestCategory.name;
    if(requestCategory.parent) category.parent = requestCategory.parent;
    if(requestCategory.level) category.level = requestCategory.level;

    let result = await category.save();
    if (result) ctx.body = response.createOKResponse(result);
    else ctx.body = response.createFailedResponse(500, 'update category fail')
});

router.get('/list', async (ctx) => {
    let parent = ctx.query.parent;
    if (!parent) {
        ctx.body = response.createFailedResponse(400, 'bad request params');
    }

    let result = await CategoryModel.find({parent: parent}).select('_id name parent level').exec();
    if (result) {
        ctx.body = response.createOKResponse(result);
    } else {
        ctx.body = response.createFailedResponse(500, 'find categories failed');
    }
});

function checkCategory(category) {
    return !(category.level > 5 || category.level < 0 || !category.level || !category.name || !category._id);
}

module.exports = router;
```
可以看到mongoose处理数据库的增删改查请求都是异步，使用es7的await语句做异步是不是非常的爽？

### 单元测试
单元测试可以帮助发现很大比例的bug，ava是一新一代的nodejs测试框架，可以异步测试（虽然这次我需要的是同步- -）具体的使用说明可以看官方github主页：https://github.com/avajs/ava
在写测试代码的时候尴尬了，我们请求接口是异步，执行测试用例也是异步，但是对category四个接口的测试我是想有顺序地执行的（比如我得先创建一个测试分类然后才能修改、查询、删除，没有顺序的话没办法每次跑单元测试都通过）。棘手的是我好像并想不到同步执行单元测试的方法- -最后还是查询官方文档得知的，在test后面加上.serial即可。看来ava还是为我们考虑到了这一点的。superkoa是基于supertest的做的一个可以让我们在ava测试代码里调用koa的框架，使用起来非常的简单。
新建一个test文件夹，添加一个test.js文件，全部的测试代码如下：
```
import test from 'ava';
import superKoa from 'superkoa';
import app from '../app';

test('hello full-stacker', async t => {
    let res = await superKoa(app).get('/');
    t.is(200, res.status);
    t.is(res.text, 'Hello full stacker!')
});

test.serial('create category', async t => {
    let testCategory = {
        _id:'test',
        name:'测试分类',
        parent:'root',
        level:1
    };
    let res = await superKoa(app)
        .post('/category')
        .send(testCategory);
    t.is(200, res.status);
    t.is(0, res.body.error);
    t.is('测试分类', res.body.data.name);
});

test.serial('update category', async t => {
    let res = await superKoa(app)
        .patch('/category')
        .send({_id: 'test', name: '测试分类2'});
    t.is(200, res.status);
    t.is(0, res.body.error);
    t.is('测试分类2', res.body.data.name);
});

test.serial('query categories by parent', async t => {
    let res = await superKoa(app)
        .get('/category/list?parent=root');
    t.is(200, res.status);
    t.is(0, res.body.error);
    t.true(res.body.data.length > 0);
});

test.serial('delete category', async t => {
    let res = await superKoa(app)
        .delete('/category?_id=test');
    t.is(200, res.status);
    t.is(0, res.body.error);
});
```
然后把package.json下的script标签下的test命令改为"test": "ava -v"
现在我们来执行下单元测试：
```
▶ npm test

> full-stacker-server@0.1.0 test /Users/judy/WeChatProjects/full-stacker/full-stacker-api
> ava -v

mount route /category.js 
mount route /index.js 

******************************************************
                MoaJS Apis Dump
******************************************************

┌─────────────────────────────────────────────────────────────────────────────┬────────┬────────────────┐
│ File                                                                        │ Method │ Path           │
├─────────────────────────────────────────────────────────────────────────────┼────────┼────────────────┤
│ /Users/judy/WeChatProjects/full-stacker/full-stacker-api/routes/category.js │ POST   │ /category/     │
├─────────────────────────────────────────────────────────────────────────────┼────────┼────────────────┤
│ /Users/judy/WeChatProjects/full-stacker/full-stacker-api/routes/category.js │ DELETE │ /category/     │
├─────────────────────────────────────────────────────────────────────────────┼────────┼────────────────┤
│ /Users/judy/WeChatProjects/full-stacker/full-stacker-api/routes/category.js │ PATCH  │ /category/     │
├─────────────────────────────────────────────────────────────────────────────┼────────┼────────────────┤
│ /Users/judy/WeChatProjects/full-stacker/full-stacker-api/routes/category.js │ GET    │ /category/list │
├─────────────────────────────────────────────────────────────────────────────┼────────┼────────────────┤
│ /Users/judy/WeChatProjects/full-stacker/full-stacker-api/routes/index.js    │ GET    │                │
└─────────────────────────────────────────────────────────────────────────────┴────────┴────────────────┘
  <-- POST /category
连接数据库成功
POST /category - 4358ms
  --> POST /category 200 4,364ms 89b
  ✔ create category (4.4s)
  <-- PATCH /category
PATCH /category - 87ms
  --> PATCH /category 200 89ms 90b
  ✔ update category
  <-- GET /category/list?parent=root
GET /category/list?parent=root - 26ms
  --> GET /category/list?parent=root 200 34ms 270b
  ✔ query categories by parent
  <-- DELETE /category?_id=test
DELETE /category?_id=test - 27ms
  --> DELETE /category?_id=test 200 28ms 90b
  ✔ delete category
  <-- GET /
GET / - 1ms
  --> GET / 200 3ms 19b
  ✔ hello full-stacker

  5 tests passed
```
看着单元测试全部通过有着莫名的快感，不知道大家是否也一样呢～
最贴一下代码地址，目前这个项目刚刚开始，也是我的nodejs试水项目。新司机上路，有问题请大家斧正。
https://github.com/ZhangYeSong/full-stacker
