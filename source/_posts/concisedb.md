---
title: concisedb
date: 2022-04-05 11:55:31
tags:
- Node.js
- Node.js 库
categories:
- 烂裤子的小经验
- 编程
- Node.js
---

最近闲着没事干，开发了一个 Node.js 数据库的库。可以做到直接存储数据到本地 JSON 文件中。用起来大概是这个样子的：

## 用法简介

```javascript
const ConciseDbSync = require('concisedb').ConciseDbSync
const JSONAdapterSync = require('concisedb').JSONAdapterSync
const path = require('path')

const adapter = new JSONAdapterSync({
  filePath: path.join(__dirname, 'db.json')
})

// 适配器 (adapter), 默认数据 （可选）, 是否需要实时更新 （默认： true）
const db = new ConciseDbSync(adapter, { test: [] })

db.data.test.push(1)

console.log(db.data) // 输出： { test: [ 1 ] }

// 尝试更改 JSON 文件的内容
setTimeout(() => {
  db.read()
  console.log(db.data) // 输出：取决于你的更改
}, 10000)
```

支持使用 `TypeScript`：

```typescript
import { ConciseDbSync, JSONAdapterSync } from 'concisedb'
import { join } from 'path'

interface Database {
  test: number[];
  username: string;
}

const init: Database = {
  test: [],
  username: 'John',
}

const adapter = new JSONAdapterSync({
  filePath: join(__dirname, 'db.json')
})

// 适配器 (adapter), 默认数据 （可选）, 是否需要实时更新 （默认： true）
const db = new ConciseDbSync(adapter, { test: [] })

db.data.test.push(1)

console.log(db.data) // 输出： { test: [ 1 ] }

// 尝试更改 JSON 文件的内容
setTimeout(() => {
  db.read()
  console.log(db.data) // 输出：取决于你的更改
}, 10000)
```

它使用了 Proxy 以实现自动更新数据到文件，并且支持异步（当然你可以不让它自动更新，具体方法请看 [README.md](https://gitee.com/rotten_lkz/concisedb/blob/main/README-zh-Hans.md/)）：

> **`db.getData()` 仍然是一个同步方法**

```javascript
const ConciseDb = require('concisedb').ConciseDb
const JSONAdapter = require('concisedb').JSONAdapter
const path = require('path')

(async () => {
  const adapter = new JSONAdapter({
    filePath: path.join(__dirname, 'db.json')
  })
  const db = new ConciseDb()
  // 方法 db.init 应该在初始化类后被调用
  // 并且需要用 await 等待方法执行完成
  // 当然，可以用 .then 代替 await
  await db.init(adapter, { test: [] })

  db.data.test.push(1)

  // db.getData() 仍然是一个同步方法
  console.log(db.data, db.getData()) // 输出： { test: [ 1 ] } { test: [ 1 ] }

  // 尝试更改 JSON 文件的内容
  setTimeout(async () => {
    await db.read()
    console.log(db.data) // 输出：取决于你的更改
  }, 10000)
})()
```

同样，支持 `TypeScript`

```typescript
import { ConciseDb, JSONAdapter } from 'concisedb'
import { join } from 'path'

(async () => {
  interface Database {
    test: number[];
    username: string;
  }

  const init: Database = {
    test: [],
    username: 'John',
  }

  const adapter = new JSONAdapter({
    filePath: join(__dirname, 'db.json')
  })
  const db = new ConciseDb()
  // 方法 db.init 应该在初始化类后被调用
  // 并且需要用 await 等待方法执行完成
  // 当然，可以用 .then 代替 await
  await db.init<Database>(adapter, init)

  db.data.test.push(1)

  // db.getData() 仍然是一个同步方法
  console.log(db.data, db.getData()) // 输出： { test: [ 1 ] } { test: [ 1 ] }

  // 尝试更改 JSON 文件的内容
  setTimeout(async () => {
    await db.read()
    console.log(db.data) // 输出：取决于你的更改
  }, 10000)
})()
```

## 版本选择

短短几天时间这个库就有 `v0` 和 `v1` 两个版本了，`v1` 主要更改是用到了适配器 (adapter)，也意味着可以通过拓展适配器来实现存储到不同地方。

当然如果想用 `v0`，我也会继续更新。`v0` 写起来看上去是这样的：

```javascript
const ConciseDb = require('concisedb')
const path = require('path')

const db = new ConciseDb(path.join(__dirname, 'db.json'), { test: [] })

db.data.test.push(1)

console.log(db.data) // 输出： { test: [ 1 ] }
```

`TypeScript` 一样支持，但是懒得写案例了。

只不过 `v0` 就支持直接存储到 JSON 文件，但是更加方便使用。

## 链接

本项目设立了官网：<https://www.concisedb.top/>（其实就是导航到 `v0` 和 `v1` 版本的文档上去）

`v0` 版本文档网址：<https://v0.concisedb.top/>

`v1` 版本文档网址：<https://v1.concisedb.top/>

[GitHub 仓库](https://github.com/Rotten-LKZ/concisedb/) | [Gitee 仓库](https://gitee.com/rotten_lkz/concisedb/) | [NPM](https://www.npmjs.com/package/concisedb)
