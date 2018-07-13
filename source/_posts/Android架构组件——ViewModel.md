---
title: Android架构组件——ViewModel
date: 2018-07-02 10:16:01
author: 
tags:
categories: android技术文档
---
### 概述
Android 官方架构组件在2017年11月份Android官方架构组件正式版发布, 并且 Google 也在 Support Library v26.1.0 以后的版本中内嵌了 Android 官方架构组件中的生命周期组件.
本文只要介绍viewModel.
ViewModel 有两个功能, 第一个功能可以使 ViewModel 以及 ViewModel 中的数据在屏幕旋转或配置更改引起的 Activity 重建时存活下来, 重建后数据可继续使用, 第二个功能可以帮助开发者轻易实现 Fragment 与 Fragment 之间, Activity 与 Fragment 之间的通讯以及共享数据...