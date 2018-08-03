---
title: 项目MVVM结构简析
date: 2018-07-26 14:44:40
author: dale.liu
tags: android技术文档
categories:
---


## MVVM框架
google官方发布了MVVM的正式库。官方的正式MVVM库主要包括下面四个：
![google_arm](/images/google_arm.png)

**其中只有ViewModel是MVVM结构中的一个组件，其他的三个都是辅助性质的。**
lifecycles 就是处理UI界面的生命周期，在26版本以后的Support库中，AppCompatActivity和SupportActivity中都实现了LifecycleOwner，内部已经对UI界面的生命周期做了处理了。
LiveData是一个抽象类，我们可以存放UI页面需要的数据，就是把数据包装在LiveData中了，我们可以观测LiveData中的数据变化，但是LiveData是跟UI的生命周期关联的，当UI页面销毁了，LiveData的数据变化回调是不会执行的。
Room 就是一个sqlite数据持久化库，我们也可以使用别的ORM库。

## MVVM架构优势
一张图看看MVVM:
![mmvm_arm](/images/mvvm_arm.jpg)

![vvvvv](/images/vvvvv.png)
看上图Model和View是不会发生关系的，ViewModel是把View和Model关联起来的.

+ View和Model双向绑定，一方的改变都会影响另一方，开发者不用再去手动修改UI的数据。互相自动更新。

+ View和Model的双向绑定是支持生命周期检测的，不会担心页面销毁了还有回调发生，这个由lifeCycle完成。

+ 不会像MVC一样Activity中代码量巨大，也不会像MVP一样出现大量的View和Presenter接口。项目结构更加低耦合。

+ 更低的耦合把各个模块分开开发，分开测试，可以分给不同的开发人员来完成。

## MVVM项目架构分析
项目整体架构如官方下图架构：
![apparcc](/images/apparcc.png)

下图是项目模块和工程之间的依赖关系：
![appmvvm](/images/appmvvm.png)


## ARouter串联各个模块
使用ARouter来跳转Activity和获取Fragment.

ARouter常见的应用场景
+ 从外部URL映射到内部页面，以及参数传递与解析
+ 跨模块页面跳转，模块间解耦
+ 拦截跳转过程，处理登陆、埋点等逻辑(添加拦截器的方法是利用Interceptor注解，实现IInterceptor接口)
+ 跨模块API调用，通过控制反转来做组件解耦

### Api常用方法
```
// 构建标准的路由请求
ARouter.getInstance().build("/home/main").navigation();

// 构建标准的路由请求，并指定分组
ARouter.getInstance().build("/home/main", "ap").navigation();

// 构建标准的路由请求，通过Uri直接解析
Uri uri;
ARouter.getInstance().build(uri).navigation();

// 构建标准的路由请求，startActivityForResult
// navigation的第一个参数必须是Activity，第二个参数则是RequestCode
ARouter.getInstance().build("/home/main", "ap").navigation(this, 5);

// 直接传递Bundle
Bundle params = new Bundle();
ARouter.getInstance()
    .build("/home/main")
    .with(params)
    .navigation();

// 指定Flag
ARouter.getInstance()
    .build("/home/main")
    .withFlags();
    .navigation();

// 获取Fragment
Fragment fragment = (Fragment) ARouter.getInstance().build("/test/fragment").navigation();

// 对象传递
ARouter.getInstance()
    .withObject("key", new TestObj("Jack", "Rose"))
    .navigation();

// 觉得接口不够多，可以直接拿出Bundle赋值
ARouter.getInstance()
        .build("/home/main")
        .getExtra();

// 转场动画(常规方式)
ARouter.getInstance()
    .build("/test/activity2")
    .withTransition(R.anim.slide_in_bottom, R.anim.slide_out_bottom)
    .navigation(this);

// 转场动画(API16+)
ActivityOptionsCompat compat = ActivityOptionsCompat.
    makeScaleUpAnimation(v, v.getWidth() / 2, v.getHeight() / 2, 0, 0);

//  makeSceneTransitionAnimation 使用共享元素的时候，需要在navigation方法中传入当前Activity
ARouter.getInstance()
    .build("/test/activity2")
    .withOptionsCompat(compat)
    .navigation();

// 使用绿色通道(跳过所有的拦截器)
ARouter.getInstance().build("/home/main").greenChannel().navigation();

// 使用自己的日志工具打印日志
ARouter.setLogger();

//获取原始的URI
String uriStr = getIntent().getStringExtra(ARouter.RAW_URI);

//关闭ARouter
ARouter.getInstance().destroy();


```
## 组件化编译和非组件化编译切换
在工程根目录下的gradle.properties文件中加入一个Boolean类型的变量，通过修改这个变量来识别编译模式：
```
# 每次更改“isBuildModule”的值后，需要点击 "Sync Project" 按钮
# isBuildModule“集成开发模式”和“组件开发模式”的切换开关
isBuildModule=false
```
然后在 各个module中的build.gradle文件中支持切换：
示例:
```
if (isBuildModule.toBoolean()) {
    apply plugin: 'com.android.application'
} else {
    apply plugin: 'com.android.library'
}
android {
    compileSdkVersion parent.ext.compileSdkVersion
    buildToolsVersion parent.ext.buildToolsVersion

    sourceSets {
        main {
            if (isBuildModule.toBoolean()) {
                manifest.srcFile 'src/main/debug/AndroidManifest.xml'
            } else {
                manifest.srcFile 'src/main/release/AndroidManifest.xml'
            }
        }
    }

    if (isBuildModule.toBoolean()) {
        packagingOptions {
            exclude 'META-INF/LICENSE'
            exclude 'META-INF/NOTICE'
            exclude 'META-INF/DEPENDENCIES'
        }
    }

    defaultConfig {
        if (isBuildModule.toBoolean()) {
            applicationId "base.android.t3t.netrequestdemo"
        }
        minSdkVersion 14
        targetSdkVersion 27
        versionCode 1
        versionName "1.0"

        javaCompileOptions {
            annotationProcessorOptions {
                arguments = [moduleName: project.getName()]
            }
        }

    }

    buildTypes {
        release {
            minifyEnabled false
            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
        }
    }

}

dependencies {
    implementation fileTree(dir: 'libs', include: ['*.jar'])
    implementation project(':basebusinesslib')
    annotationProcessor "com.alibaba:arouter-compiler:$arouter_compiler_version"

}

```

