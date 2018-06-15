---
title: 如何写第一篇hexo博客
date: 2018/6/14 17:10:00
author: TFsky
tags: 
	- hexo
	- 教程
categories: 
	- 教程	
    - hexo 
---
![hello-hexo-header](/images/hello_hexo.jpg)

在准备写博客之前可阅读一下简单教程

### 须知

 本站博客使用hexo搭建，博客相关资源存在 https://github.com/T3Team/T3Team.github.io 上。

- hexo分支为默认分支，存放hexo网站资源

- master分支存放静态文件

### 初始化/恢复

重装电脑后，或者在其它电脑上想修改博客：

1. 安装 git；
2. 安装 Nodejs 和 npm；
3. 使用 `git clone git@github.com:T3Team/T3Team.github.io.git` 将仓库拷贝至本地；
4. 在文件夹内执行以下命令 `npm install hexo-cli -g`、`npm install`、`npm install hexo-deployer-git`。
5. 执行 hexo g 、hexo s 然后打开浏览器，输入地址http://localhost:4000/ ,即可看到效果

### 新建文章

##### 命令方式

```
$ hexo new test    //test为文章名 
```

.md文件头部会自动生成如下内容
- title 文章标题
- date 文章创建时间
- author 作者，不填则显示默认值（本博客配置为T3team）
- tags 标签
- categories 文章分类

```
---
title: test
date: 2018/6/14 17:10:00
author: 
tags: 
categories: 
---
```

本文头部参数如下

```
---
title: 如何写第一篇hexo博客
date: 2018/6/14 17:10:00
author: TFsky
tags: 
	- hexo
	- 教程
categories: 
    - hexo 
    - 教程
---
```

此时会在`source/_posts`目录下生成`test.md`文件，输入些许内容，然后保存.

生成下，看看效果

```
$ hexo clean
$ hexo g
$ hexo s
```

访问 **localhost:4000** 即可

##### 直接方式

在 **source/_posts/**下新建一个`.md`文件也可



### 发布文章

在本地对博客修改（包括修改主题样式、发布新文章等）后：

1. 依次执行 `git add`、`git commit -m "xxxx" (简单的描述下信息)` 和 `git push origin hexo` 来提交 hexo 网站源文件；

2. 执行 `hexo g -d` 生成静态网页部署至 Github 上。

### 相关参考文章:

- [Hexo官方文档](https://hexo.io/zh-cn/docs/)

- [Hexo 博客搭建指南](https://github.com/limedroid/HexoLearning)

- [Hexo博客备份](https://blog.itswincer.com/posts/7efd2818/)           