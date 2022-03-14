---
title: Node.js 配置别名 alias 的两种方法
date: 2020-10-24 18:41:36
tags:
- Node.js
categories:
- 裤子的小经验
- 编程
- Node.js
---

什么是别名？

```typescript
// import xxx from '../../../xxx';
import xxx from '@/xx/xx/xxx';
```

可以省去 `../../../xxx` ，直接从配置的目录开始找文件

## 环境准备

 - [Node.js（此文章使用v12.16.3版本）](https://nodejs.org/en/download/)
 - [Visual Studio Code（非必须 此文章使用1.50.1版本）](https://code.visualstudio.com/)

> 1. Node.js如果下载过慢，可以从[淘宝镜像](https://npm.taobao.org/mirrors/node/)下载

## 第一种：[webpack](https://webpack.js.org/)

使用 webpack 进行打包并且在 ``webpack.config.js`` 配置里面配置 alias

配置 ``webpack.config.js``

```javascript
const path = require('path');
const resolve = path.resolve;

module.exports = {
  entry: './src/index.js',             // 入口文件
  output: {
    filename: 'bundle.js',			   // 输出文件名
    path: resolve(__dirname, 'dist')   // 输出文件路径
  },
  resolve: {
    alias: {
      '@': resolve(__dirname, 'src')   // 别名设置
    }
  }
}
```

之后在目录 ``src`` 下面创建 ``index.js`` 和 ``test.js``

``index.js``

```javascript
import test from '@/test';	// 这里的 @/xxx 相当于 src/xxx
console.log(test);
```

``test.js``

```javascript
export default 'test';	// 随便导出什么内容
```

之后控制台输入 ``webpack`` 编译之后运行 ``node dist/bundle.js`` 即可输出结果 ``test``

## 第二种：[module-alias](https://github.com/ilearnio/module-alias)

控制台输入 ``npm install module-alias`` 或 ``yarn add module-alias`` 安装 module-alias

然后进入 ``package.json`` 加入如下配置：

```json
"_moduleAliases": {
  "@": "src"
}
```

（如果使用 TypeScript 的话，可以用 ``tsc`` 直接编译。但是清注意这里的 ``src`` 应改成打包后的目录比如 ``dist`` ``build`` 等）

之后在主入口文件头部引用 ``module-alias/register`` 如：

``index.js``

```javascript
require('module-alias/register');
const test = require('@/test');
console.log(test);
```

``test.js``

```javascript
module.exports = 'test';
```

之后控制台运行 ``node src/index.js`` 即可输出结果 ``test``

## IDE 路径提示设置

虽然打包运行可以通过别名，但是 IDE 并不认识

我们需要在项目根目录新建文件 ``jsconfig.json`` （使用 TypeScript 则是 ``tsconfig.json`` ）
在 ``compilerOptions`` 加入：

```json
"baseUrl": "./src",
"paths": {
  "@/*": ["*"]
}
```

文件最后如下：

```json
{
  "compilerOptions": {
    "baseUrl": "./src",
    "paths": {
      "@/*": ["*"]
    }
  }
}
```

## 总结

安装环境之后两种办法：

1. 修改 ``webpack`` 配置以支持别名打包
2. 使用 ``module-alias`` 库进行编译运行