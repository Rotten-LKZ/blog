---
title: 搭建属于自己的博客
date: 2022-03-13 13:34:39
categories:
- 裤子的小教程
- 编程
tags: 
- hexo
- Node.js
- 教程
- 编程
---

## 为什么要有属于自己的博客？

市面上有非常多的平台供我们写博客，比如：简书、CSDN、掘金 等……

所以作为一个有理想有愿望的新青年，为什么要有一个属于自己的博客呢？

没错，就是爱折腾！！！

哈哈哈，这就是这个博客的诞生目的之初。

## 为什么用 hexo？

其实我刚开始用博客的时候是 wordpress，但是感觉用的不爽 ~~（不够折腾）~~。索性就用静态博客 hexo 了。

本来说还想考虑 typecho 的，但是好像说正式版很久没有更新了并且配置起来比 wordpress 麻烦（虽然 hexo 配置起来也不简单就是了），所以就放弃使用 typecho 而转用 hexo 博客了。

**更主要的是**

[云云酱](https://www.yunyoujun.cn/) 有写过一个 hexo 博客的主题 [Hexo-Theme-Yun](https://yun.yunyoujun.cn/)，所以就拿来用了！！！

大爱云云（不）

## 具体配置

1. 首先，你要去 [hexo](https://hexo.io/zh-cn/docs/index.html) 官网按照教程安装 hexo

2. 去云云酱的 [Hexo-Theme-Yun](https://yun.yunyoujun.cn/)，跟着教程一步一步安装并且跟着里面的文档配置

3. 我这里用的是 GitHub actions 自动打包然后用 FTP 上传到服务器上的。其路径是 `.github/workflows/deploy.yml`（没有对应文件夹或文件请新建）。GitHub actions 具体配置如下：

```yml
name: Deploy

on:
  push:
    branches:
      - main

jobs:
  build:
    name: 🍳 Build on node ${{ matrix.node_version }} and ${{ matrix.os }}
    runs-on: ubuntu-latest
    strategy:
      matrix:
        os: [ubuntu-latest]
        node_version: [16.x]

    steps:
      - name: 🤔 Checkout
        uses: actions/checkout@v2
      
      - name: 🚚 Install dependencies
        run: |
          npm install

      - name: 🎉 Deploy hexo
        run: |
          npm run deploy

      - name: 📂 Sync files
        uses: SamKirkland/FTP-Deploy-Action@4.3.0
        with:
          server: ${{ secrets.ftp_server }}
          username: ${{ secrets.ftp_username }}
          password: ${{ secrets.ftp_password }}
          local-dir: ./public/
          server-dir: /wwwroot/lkzstudio/www/
```

*注：最后几行那三个参数是在你 GitHub 仓库里 Settings -> Secrets -> Actions -> New repository secret 添加的*

![Secrets of GitHub actions](https://cdn.jsdelivr.net/gh/Rotten-LKZ/cdn@main/images/content/github-actions-secrets-1370ca.png)

4. 之后把你的项目用 `git push` 提交到服务器，GitHub 就会自动执行 GitHub Actions 然后上传到你的服务器了

5. 解析域名并绑定 FTP 上传的服务器，一切就大功告成了！
