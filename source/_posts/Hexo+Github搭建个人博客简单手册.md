---
title: Hexo博客搭建简单教程
categories: 
  - Hexo
tags: 
  - Hexo
date: 2017-12-04 14:36:21
---

近期想多点备份自己的笔记，就盯上了Github+Hexo这种博客模式。 我想Github的数据应该不可能丢失的。

## 准备阶段

你需要准备以下环境：

 - Git
 - Node.js
 - Github 账户

### Git 环境准备
1. 访问Git 官网下载安装包，一路Next
2. 验证安装成功
```bash
git --version
```
推荐安装SourceTree用来管理git。
3. 配置SSH 访问Github

### Node.js 环境准备
1. 访问Node.js 官网下载安装包，一路Next安装即可。
2. 验证安装成功
```bash
node -v
npm -v
```
出现版本信息即为安装成功。

### GitHub 账户准备
1. 访问github，注册帐号
2. 创建代码库`New repository`
    1) 在Repository name下填写yourname.github.io
    2) 创建
3. 开启github page 功能
    1）点击界面右侧的`Settings`
    2）下拉至GitHub Pages，确认是否开启，未开启则开启


----------


## 安装Hexo

到这一步，我们就可以安装Hexo了。
1. 执行安装命令`npm install hexo-cli -g`
2. `npm install hexo --save`
3. `hexo -v`

## 初始化Hexo
接着上面的命令
1. `hexo init`
2. `npm install`
3. 如果后面想用git部署，`npm install hexo-deployer-git --save`

## 启动Hexo
1. 使用`hexo g` 生成静态页面
2. 命令`hexo s -p 4455`，启动服务器，`-p 4455`设置端口，可以省略

## 使用Hexo
### 修改全局配置文件


|  参数  |  描述  |
| --------   | ----- |
|  title  |  网站标题  |
|  subtitle  |  网站副标题  |
|  description  |  网站描述  |
|  author  |  您的名字  |
|  language  |  网站使用的语言  |
|  timezone  |  网站时区。Hexo 默认使用您电脑的时区。时区列表。比如说：America/New_York, Japan, 和 UTC 。  |

## 部署Hexo
前面如果设置过SSH
直接执行命令
```bash
hexo g
hexo d
```
即可部署到github上。

----------


## 利用Github分支备份Hexo博客源文件
### 场景
Hexo 部署博客很方便，我的这个博客也是用 Hexo 部署在 GitHub Pages 上的，有得人可能在多台电脑上写博客，这个时候需要把博客的源文件备份在一个地方，这样只需把博客源文件复制下来就可以在另一个地方写博客并部署到 GitHub Pages上了

本篇介绍的就是利用博客的 repo 分支（ master 分支的必须用来存放你博客网站文件）托管 Hexo 源文件和配置达到备份的目的，下面开始正题:

1. 把博客目录的源文件push到repo分支上
```
git init
git add .
git commit -m "commit first time"
git remote add origin https://github.com/your-name/your-name.github.io.git
```

接下来就是把Hexo源文件 push 上去，但是关键的地方到了，master上是 Hexo 生成博客网页的代码，而我们 Hexo 源文件是要 push 到一个分支上面的，所以接下来先要在 repo 上新建一个分支
新建一个叫做hexoSource的分支：

```
git branch blogSource
```
查看本地分支，并且切换到 blogSource 分支
```
git branch
git checkout blogSource
```
然后拉取远程代码，再把刚才添加的 Hexo 源文件代码 push 到blogSource这个分支：

```
git pull origin master
git push -u origin blogSource
```
然后就可以在 repo 上看到分支里面已经有博客的源文件了

只要维护你的md文件在分支里`blogSource`就可以了。


----

## 问题

1. Q: hexo new page tags 页面内容为空 
开启/tags路由后，访问页面内容仅显示两个字：标签，HTML如下

**标签**

请问是什么原因？

另外，theme下的layout，category.ejs,tag.ejs是如何引用的？谢谢！

A: 发现主题目录下有一个[“_source”文件夹]（它里面的内容正是about,categories,tags分别已写好的index.md）。复制到[Hexo根目录/source文件夹]内覆盖，就可以了。