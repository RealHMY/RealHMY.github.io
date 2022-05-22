---
title: 半小时内搭建自己的博客
date: 2022-05-22 14:14:59
author: RealHMY
tags:
---

## 简介

该教程基于 GitHub，搭建一个 Hexo 博客。

环境：Node.js \ Git \ VSCode

## 创建新仓库

点击右上角的加号，新建仓库。

![](/images/test.png)

仓库名字格式：`账号名.github.io`

![](/images/2022-05-22-14-41-59.png)

由于我已经创建了仓库，所以会提示仓库已存在。填入仓库名称后，点击 Create repository，完成仓库的创建。

## 拉取项目到本地

VSCode 上新建窗口。

![](/images/2022-05-22-14-48-37.png)

选择克隆存储库

![](/images/2022-05-22-14-49-42.png)

将 GitHub 的仓库链接复制上去即可。

![](/images/2022-05-22-14-52-09.png)

## 搭建 Hexo 项目

在 D 盘创建空的文件夹，准备 Hexo 的项目。

打开 cmd 或 powershell，跳转到 D 盘的空文件夹下。

安装 Hexo。

```npm
npm install -g hexo-cli
```

初始化 Hexo 项目。

```npm
hexo init
```

安装必备的组件。

```npm
npm install
```

新建完成后，指定文件夹Hexo目录下有：

* node_modules: 依赖包
* public：存放生成的页面
* scaffolds：生成文章的一些模板
* source：用来存放你的文章
* themes：主题
* _config.yml: 博客的配置文件

测试项目是否搭建成功。

方法一：

```npm
hexo g
hexo s
```

`hexo g` 是 `hexo generate` 的缩写
`hexo s` 是 `hexo server` 的缩写

方法二：

```npm
npm run server
```

以上两种方法效果一样。如果要停止，按下快捷键 `CTRL + C`。

## 添加 SSH 到 Github

创建 SSH。

```npm
ssh-keygen -t rsa -C "youremail"
```

以记事本的方式打开 id_rsa.pub 文件。复制所有文本。

打开 Github，点击头像，打开设置。

![](/images/2022-05-22-15-10-05.png)

点击添加新的 SSH key。

![](/images/2022-05-22-15-11-08.png)

title 推荐使用英文，然后粘贴 id_rsa.pub 文件的所有文本到 key 上。

![](/images/2022-05-22-15-13-13.png)

## 将 Hexo 项目推送到 Github 上

我们要准备两个分支：主分支存储源代码，次分支存储静态 HTML 文件。

我的主分支是 main，次分支是 blog。之前拉取的分支是主分支，所以要预先创建好空白的次分支。

将 D 盘的 Hexo 项目下的所有文件复制到 vscode 上。

![](/images/2022-05-22-15-20-55.png)

主分支可以提交，也可以不提交，因为博客的展示只需要次分支，主分支只保存源代码。

设置要部署的分支为次分支。

![](/images/2022-05-22-15-27-13.png)

选中次分支，并保存。

![](/images/2022-05-22-15-28-14.png)

打开 _config.yml，编辑最后一行。

```yaml
deploy:
  type: git
  repository: https://github.com/你的账号名/你的账号名.github.io.git
  branch: 你的次分支名称
```

安装 deploy-git。

```npm
npm install hexo-deployer-git --save
```

部署到 Github 上。

```npm
hexo clean
hexo generate
hexo deploy
```

在浏览器输入 `你的账号名.github.io` 查看自己的博客

![](/images/2022-05-22-15-29-50.png)
