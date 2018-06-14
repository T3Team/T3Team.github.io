---
title: 如何写第一篇hexo博客
tags: 
	- hexo
	- 教程
categories: 
	- 教程	
---
在准备写博客之前可阅读一下简单教程

## 须知

 本站博客使用hexo搭建，博客相关资源存在 https://github.com/T3Team/T3Team.github.io 上。

- hexo分支为默认分支，存放hexo网站资源
-  master分支存放静态文件

### 恢复

重装电脑后，或者在其它电脑上想修改博客：

1. 安装 git；
2. 安装 Nodejs 和 npm；
3. 使用 `git clone git@github.com:T3Team/T3Team.github.io.git` 将仓库拷贝至本地；
4. 在文件夹内执行以下命令 `npm install hexo-cli -g`、`npm install`、`npm install hexo-deployer-git`。

### 修改

在本地对博客修改（包括修改主题样式、发布新文章等）后：

1. 依次执行 `git add`、`git commit -m ""` 和 `git push origin hexo` 来提交 hexo 网站源文件；

2. 执行 `hexo g -d` 生成静态网页部署至 Github 上。

   

相关参考文章:

​            [Hexo官方文档](https://hexo.io/zh-cn/docs/)

​           [Hexo 博客搭建指南](https://github.com/limedroid/HexoLearning)

​           [Hexo博客备份](https://blog.itswincer.com/posts/7efd2818/)           