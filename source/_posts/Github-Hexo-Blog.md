---
title: 在Github上建立Blog
date: 2017-01-06 13:17:11
categories: 
- Hexo
tags: 
- Github 
- Hexo 
---


技术是需要收纳的，利用Hexo在Github上建立个人博客，收纳自己的技术。

<!--more-->

# 工具安装
## 原料
[Github](http://github.com) [Git](https://www.git-scm.com/download) [nodejs](https://nodejs.org/en/) [Hexo](https://hexo.io/)

## Github
[Github官网](http://github.com)注册一个自己的账户

## Git
[Git下载页面](https://www.git-scm.com/download)

## Node.js
时下最流行的工具npm  [官网](https://nodejs.org/en/)下载

## Hexo
1. 新建一个供Hexo使用的目录，path当然尽量全英文(比如d:/hexo)
2. 进入目录，鼠标右键打开Git Bash 在命令行中输入以下指令
3. 我们在这里就可以试一下npm了。 以下这些指南在[Hexo官网](https://hexo.io/)可查。
```bash
		npm install hexo-cli -g
		hexo init blog
		cd blog
		npm install
		hexo server
```
如果命令行没有报错，你应该能看到Hexo默认主页[http://localhost:4000](http://localhost:4000)


# Github + Hexo
接下来 选一款还不错的主题部署到自己的Github上吧
* [Cover](https://github.com/daisygao/hexo-themes-cover) - A chic theme with facebook-like cover photo 
* [Oishi](https://github.com/henryhuang/oishi) - A white theme based on Landscape plus and Writing. 
* [TKL](https://github.com/SuperKieran/TKL) - A responsive design theme for Hexo. 
* [Tinnypp](https://github.com/levonlin/Tinnypp) - A clean, simple theme based on Tinny 
* [Writing](https://github.com/yunlzheng/hexo-themes-writing) - A small and simple hexo theme based on Light 
* [Yilia](https://github.com/litten/hexo-theme-yilia) - Responsive and simple style 
* [Pacman voidy](https://github.com/Voidly/pacman) - A theme with dynamic tagcloud and dynamic snow

## 本地部署
1. 选择一个主题，fork它
2. 到自己的branch下，把repo的名字改成 ur_name.github.io  ur_name = 你github的账户名
3. 下载你的主题到本地
```bash
		git clone https://github.com/SuperKieran/TKL themes/theme_name  
```
4. 在_config.yml文件中找到，改成选择的主题名
```bash
		theme: theme_name
```
5. 打开git bash 启动Hexo
```bash
		hexo server
```
6. 打开本地主页[http://localhost:4000](http://localhost:4000) 是不是已经换成新的主题了呢

## 部署到Github
1. 修改配置文件 在_config.yml文件中找到Deployment部分，改动如下
```bash
		deploy:
		 type: git
		 repo: https://github.com/ur_name/ur_name.github.io.git
		 branch: master
```

2. 打开git bash 部署
```bash
		hexo deploy
```

3. 打开你的主页[ur_name.github.io](http://ur_name.github.io) 这样，你的博客就搭建好啦

# 日常维护
当然，你需要研究主题模板的架构，慢慢把里面的东西都改成属于自己的

## 新建文章
1. 打开git bash，
```bash
		hexo new "paper_name"
```
2. source/_posts目录下面，多了一个paper_name.md的文件。打开之后我们会看到
```bash
		---
		title: new article 
		date: 2017-01-06 13:17:11
		tags: 
		---
```

在这里可以修改标题title 标签tags 分类categories
当然，这个文件的内容需要符合MarkDown的格式

## Hexo日常部署
```bash
	hexo clean
	hexo generate
	hexo deploy
```
依次为 清除、生成、上传
deploy如果失败，运行
```bash
Error Code: ERROR Deployer not found: git
Run: npm install --save hexo-deployer-git
```

## Hexo 源文件&主题 备份
安装hexo-git-backup
```bash
npm install --save hexo-git-backup
```
同时需要在_config.yml中添加：
```yml
backup:
    type: git
    theme: TKL
    repository:
       github: git@github.com:Sidamor/hexo-git-backup.git
```


## 更换主题
1. 下载你的主题到本地
```bash
		git clone https://github.com/ur_name/ur_name.github.io.git themes/theme_name  
```
2. 在_config.yml文件中找到，改成选择的主题名
```bash
		theme: theme_name
```
3. 打开git bash 启动Hexo
```bash
		hexo server
```
## 使用本地图片
解决方案：CodeFalling/hexo-asset-image 
1. 首先确认 _config.yml 中有 post_asset_folder:true 。 
2. 在 hexo 目录，执行
	npm install hexo-asset-image --save
3. 在source/_post中，找到文章md所在目录，建立文章同名目录，将图片放入新建目录中。
4. 只要使用
```bash
	![name](文章名/Pic.png) 
```
就可以插入图片。 	

## 其他功能参考链接
[HEXO使用CDN加速并优化SEO和代码着色](http://tyr.gift/hexo-optimization.html)
[HEXO为主题插入文章目录和多说评论](http://tyr.gift/add-toc-for-hexo.html)
