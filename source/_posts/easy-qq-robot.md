---
title: 简单的 QQ 机器人
date: 2022-03-29 12:24:15
tags:
- QQ 机器人
- Node.js
categories:
- 烂裤子的小经验
- 编程
- Node.js
---

## 介绍

基于 [mirai](https://github.com/mamoe/mirai) 和 [mirai-ts](https://github.com/YunYouJun/mirai-ts) 开发的 QQ 机器人。

***由于基于 [mirai](https://github.com/mamoe/mirai)，所以开源协议为 [AGPL-3.0 License](https://www.gnu.org/licenses/agpl-3.0.en.html)。请尊重开源协议。***

## 配置

1. 在 <https://github.com/mamoe/mirai/blob/dev/docs/UserManual.md> 下载想要的版本（建议控制台版本，这里用控制台版本讲解）

2. 下载下来后运行，它会问你要不要额外装 java 以及是否安装。是否安装 java 看你自己，第二个问题就选择 Y 即可。过会儿本地将会出现好几个文件/文件夹

3. 建议打开控制台到安装目录，输入 `./mcl`

4. 输入 `exit` 或者按 `Ctrl + C` 结束运行

5. 打开控制台到安装目录，输入 `./mcl --update-package net.mamoe:mirai-api-http --channel stable-v2 --type plugin`

6. 再次输入 `./mcl`，如果出现 `net.mamoe:mirai-api-http` 安装失败，请访问 <https://github.com/project-mirai/mirai-api-http/releases> 下载最新版本 `jar` 文件，放入 `plugins` 文件夹即可。

7. 下载 zip 或者 clone 我的项目：[easy-robot](https://github.com/Rotten-LKZ/easy-robot)，按照 `README.md` 运行程序即可

是不是非常简单？

具体功能就看 GitHub 上面 [`README.md`](https://github.com/Rotten-LKZ/easy-robot/blob/main/README.md) 就好了，在博客上就不更新了。
