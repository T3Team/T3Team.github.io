---
title: 基于glide框架的图片加载器简介
date: 2018-07-30 09:27:09
author: dale.liu
tags: android技术文档
categories: android技术文档
---


背景
=============
 - 为什么选择Glide？
  http://blog.csdn.net/github_33304260/article/details/70213300
 - Glide直接能使用了，已经很方便了，为什么还要封装？
   - 入口统一，所有图片加载都在这一个地方管理，一目了然，即使有什么改动我也只需要改这一个类就可以了。
   - 虽然现在的第三方库已经非常好用，但是如果我们看到第三方库就拿来用的话，很可能在第三方库无法满足业务需求或者停止维护的时候，发现替换库，工作量可见一斑。这就是不封装在切库时面临的窘境！
   - 外部表现一致，内部灵活处理原则
   - 更多内容参考：[如何正确使用开源项目？](http://mp.weixin.qq.com/s?__biz=MzA4NTQwNDcyMA==&mid=2650661623&idx=1&sn=ab28ac6587e8a5ef1241be7870851355#rd)  




## Glide基本使用
Glide使用一个流接口（Fluent Interface）。用Glide完成一个完整的图片加载功能请求，需要向其构造器中至少传入3个参数，分别是：
> - with(Context context)- Context是许多Android API需要调用的， Glide也不例外。这里Glide非常方便，你可以任意传递一个Activity或者Fragment对象，它都可以自动提取出上下文。
> - load(String imageUrl) - 这里传入的是你要加载的图片的URL，大多数情况下这个String类型的变量会链接到一个网络图片。
> -  into(ImageView targetImageView) - 将你所希望解析的图片传递给所要显示的ImageView。

example:

```
ImageView targetImageView = (ImageView) findViewById(R.id.imageView);
String internetUrl = "http://i.imgur.com/DvpvklR.png";

Glide
    .with(context)
    .load(internetUrl)
    .into(targetImageView);
```

## 常用API
> - **thumbnail(float sizeMultiplier)**. 请求给定系数的缩略图。如果缩略图比全尺寸图先加载完，就显示缩略图，否则就不显示。系数sizeMultiplier必须在(0,1)之间，可以递归调用该方法。
> - ** (DiskCacheStrategy strategy).**设置缓存策略。> -DiskCacheStrategy.SOURCE：缓存原始数据，DiskCacheStrategy.RESULT：缓存变换(如缩放、裁剪等)后的资源数据，DiskCacheStrategy.NONE：什么都不缓存，DiskCacheStrategy.ALL：缓存SOURC和RESULT。默认采用> -> -DiskCacheStrategy.RESULT策略，对于download only操作要使用> -DiskCacheStrategy.SOURCE。
> - **priority(Priority priority)**. 指定加载的优先级，优先级越高越优先加载，但不保证所有图片都按序加载。枚举Priority.IMMEDIATE，Priority.HIGH，Priority.NORMAL，Priority.LOW。默认为Priority.NORMAL。
> - **dontAnimate()** 移除所有的动画。
> - **animate(int animationId).** 在异步加载资源完成时会执行该动画。
> - **animate(ViewPropertyAnimation.Animator animator).** 在异步加载资源完成时会执行该动画。
> - **animate(Animation animation).** 在异步加载资源完成时会执行该动画。
> - **placeholder(int resourceId)**. 设置资源加载过程中的占位Drawable。
> - **error(int resourceId).** 设置load失败时显示的Drawable。
> - **skipMemoryCache(boolean skip).** 设置是否跳过内存缓存，但不保证一定不被缓存（比如请求已经在加载资源且没设置跳过内存缓存，这个资源就会被缓存在内存中）。
> - **override(int width, int height).** 重新设置Target的宽高值（单位为pixel）。
> - **into(Y target)**.设置资源将被加载到的Target。
> - **into(ImageView view).** 设置资源将被加载到的ImageView。取消该ImageView之前所有的加载并释放资源。
> - **into(int width, int height)**. 后台线程加载时要加载资源的宽高值（单位为pixel）。
> - **preload(int width, int height)**. 预加载resource到缓存中（单位为pixel）。
> - **asBitmap().** 无论资源是不是gif动画，都作为Bitmap对待。如果是gif动画会停在第一帧。
> - **asGif().** 把资源作为GifDrawable对待。如果资源不是gif动画将会失败，会回调.error()。

***更多Glide详细介绍可以看[Glide官网](https://github.com/bumptech/glide)以及[Glide教程系列文章](http://www.jianshu.com/p/7610bdbbad17)***

如何封装
=============

封装后的基本使用样式：

```
ImageLoader.with(this)
	.url("http://img.yxbao.com/news/image/201703/13/7bda462477.gif")
	.placeHolder(R.mipmap.ic_launcher,false)
	.rectRoundCorner(30, R.color.colorPrimary)
	.blur(40)
	.into(iv_round);
```

只需要关心ImageLoader就好了，就算里面封装的库更换、更新也没关系，因为对外的接口是不变的。实际操作中是由实现了ILoader的具体类去操作的，这里我们只封装了GlideLoader，其实所有操作都是由ImageLoader下发指令，由GlideLoader具体去实现的。这里如果想封装别的第三方库，只需要实现ILoader自己去完成里面的方法。

##初始化

```
	public static int CACHE_IMAGE_SIZE = 250;

    public static void init(final Context context) {
        init(context, CACHE_IMAGE_SIZE);
    }

    public static void init(final Context context, int cacheSizeInM) {
        init(context, cacheSizeInM, MemoryCategory.NORMAL);
    }

    public static void init(final Context context, int cacheSizeInM, MemoryCategory memoryCategory) {
        init(context, cacheSizeInM, memoryCategory, true);
    }

    /**
     * @param context        上下文
     * @param cacheSizeInM   Glide默认磁盘缓存最大容量250MB
     * @param memoryCategory 调整内存缓存的大小 LOW(0.5f) ／ NORMAL(1f) ／ HIGH(1.5f);
     * @param isInternalCD   true 磁盘缓存到应用的内部目录 / false 磁盘缓存到外部存
     */
    public static void init(final Context context, int cacheSizeInM, MemoryCategory memoryCategory, boolean isInternalCD) {
        ImageLoader.context = context;
        GlobalConfig.init(context, cacheSizeInM, memoryCategory, isInternalCD);
    }
```

从这里可以看出我们提供了四个构造器，这里注释详细说明了所有参数的用法及意义。

##你所关心的类--ImageLoader
ImageLoader是封装好所有的方法供用户使用的，让我们看看都有什么方法：
> - ImageLoader.init(Context context) //初始化
> - ImageLoader.trimMemory(int level); 
> - ImageLoader.clearAllMemoryCaches();
> - ImageLoader.getActualLoader(); //获取当前的loader 
> - ImageLoader.with(Context context) //加载图片
> - ImageLoader.saveImageIntoGallery(String url) // 保存图片到相册
> - ImageLoader.pauseRequests() //取消请求
> - ImageLoader.resumeRequests() //回复的请求（当列表在滑动的时候，调用pauseRequests()取消请求，滑动停止时，调用resumeRequests()恢复请求  等等）
> - ImageLoader.clearDiskCache()//清除磁盘缓存(必须在后台线程中调用)
> - ImageLoader.clearMomoryCache(View view) //清除指定view的缓存
> - ImageLoader.clearMomory() // 清除内存缓存(必须在UI线程中调用)


##图片的各种设置信息--SingleConfig
我们所设置图片的所有属性都写在这个类里面。下面我们详细的看一下：

> - url(String url) //支持filepath、图片链接、contenProvider、资源id四种
> - thumbnail(float thumbnail)//缩略图
> - rectRoundCorner(int rectRoundRadius, int overlayColorWhenGif) //形状为圆角矩形时的圆角半径
> - asSquare() //形状为正方形
> - colorFilter(int color) //颜色滤镜
> - diskCacheStrategy(DiskCacheStrategy diskCacheStrategy) 
    1. DiskCacheStrategy.ALL 使用DATA和RESOURCE缓存远程数据，仅使用RESOURCE来缓存本地数据。
    2. DiskCacheStrategy.NONE 不使用磁盘缓存
    3. DiskCacheStrategy.DATA 在资源解码前就将原始数据写入磁盘缓存
    4. DiskCacheStrategy.RESOURCE 在资源解码后将数据写入磁盘缓存，即经过缩放等转换后的图片资源。
    5. DiskCacheStrategy.AUTOMATIC 根据原始图片数据和资源编码策略来自动选择磁盘缓存策略。（默认）
> - asCircle(int overlayColorWhenGif)//加载圆形图片
> - placeHolder(int placeHolderResId) //占位图
> - override(int oWidth, int oHeight) //加载图片时设置分辨率 a
> - scale(int scaleMode) // CENTER_CROP等比例缩放图片，直到图片的狂高都大于等于ImageView的宽度，然后截取中间的显示 ; FIT_CENTER 等比例缩放图片，宽或者是高等于ImageView的宽或者是高 默认：FIT_CENTER
> - animate(int animationId ) 引入动画
> - animate( Animation animation) 引入动画
> - animate(ViewPropertyAnimation.Animator animato) 引入动画
> - asBitmap(BitmapListener bitmapListener)// 使用bitmap不显示到imageview
> - into(View targetView) //展示到imageview
> - colorFilter(int filteColor) //颜色滤镜
> - blur(int blurRadius) ／/高斯模糊
> - brightnessFilter(float level) //调节图片亮度
> - grayscaleFilter() //黑白效果
> - swirlFilter() //漩涡效果
> - toonFilter() //油画效果
> - sepiaFilter() //水墨画效果
> - contrastFilter(float constrasrLevel) //锐化效果
> - invertFilter() //胶片效果
> - pixelationFilter(float pixelationLevel)  //马赛克效果
> - sketchFilter() //  //素描效果
> - vignetteFilter() //晕映效果

##中转站--GlideLoader 
GlideLoader实现ILoader接口。在使用的时候我们虽然不用关心这个类。

```
public class GlideLoader implements ILoader {

    /**
     * @param context        上下文
     * @param cacheSizeInM   Glide默认磁盘缓存最大容量250MB
     * @param memoryCategory 调整内存缓存的大小 LOW(0.5f) ／ NORMAL(1f) ／ HIGH(1.5f);
     * @param isInternalCD   true 磁盘缓存到应用的内部目录 / false 磁盘缓存到外部存
     */
    @Override
    public void init(Context context, int cacheSizeInM, MemoryCategory memoryCategory, boolean isInternalCD) {
        Glide.get(context).setMemoryCategory(memoryCategory); //如果在应用当中想要调整内存缓存的大小，开发者可以通过如下方式：
        GlideBuilder builder = new GlideBuilder();
        if (isInternalCD) {
            builder.setDiskCache(new InternalCacheDiskCacheFactory(context, cacheSizeInM * 1024 * 1024));
        } else {
            builder.setDiskCache(new ExternalPreferredCacheDiskCacheFactory(context, cacheSizeInM * 1024 * 1024));
        }

    }

    @Override
    public void request(final SingleConfig config) {
        RequestOptions requestOptions = getRequestOptions(config);//得到初始的 RequestOptions

        RequestBuilder requestBuilder = getRequestBuilder(config); //得到一个正确类型的 RequestBuilder(bitmap or 其他加载)

        if (requestBuilder == null) {
            return;
        }

        requestBuilder.apply(requestOptions);//应用RequestOptions


        //设置缩略图
        if (config.getThumbnail() != 0) { //设置缩略比例
            requestBuilder.thumbnail(config.getThumbnail());
        }

        //设置图片加载动画
        setAnimator(config, requestBuilder);

        if (config.isAsBitmap()) {//如果是获取bitmap,则回调
            SimpleTarget target = new SimpleTarget<Bitmap>(config.getWidth(), config.getHeight()) {
                @Override
                public void onResourceReady(@NonNull Bitmap resource, @Nullable Transition<? super Bitmap> transition) {
                    if (config.getBitmapListener() != null) {
                        config.getBitmapListener().onSuccess(resource);
                    }
                }

            };
            requestBuilder.into(target);
        } else {//如果是加载图片，（无论是否为Gif）
            if (config.getTarget() instanceof ImageView) {
                requestBuilder.into((ImageView) config.getTarget());
            }
        }

    }

    private RequestOptions getRequestOptions(SingleConfig config) {
        RequestOptions options = new RequestOptions();
        //设置磁盘缓存
        if (config.getDiskCacheStrategy() != null) {
            options.diskCacheStrategy(config.getDiskCacheStrategy());
        }else{
            options.diskCacheStrategy(DiskCacheStrategy.AUTOMATIC);//默认为自动选择
        }
        if (ImageUtil.shouldSetPlaceHolder(config)) {
            options = options.placeholder(config.getPlaceHolderResId());
        }

        int scaleMode = config.getScaleMode();

        switch (scaleMode) {
            case ScaleMode.CENTER_CROP:
                options.centerCrop();
                break;
            case ScaleMode.FIT_CENTER:
                options.fitCenter();
                break;
            default:
                options.fitCenter();
                break;
        }

        //设置图片加载的分辨 sp
        if (config.getoWidth() != 0 && config.getoHeight() != 0) {
            options.override(config.getoWidth(), config.getoHeight());
        }


        //设置图片加载优先级
        setPriority(config, options);

        if (config.getErrorResId() > 0) {
            options.error(config.getErrorResId());
        }
        setShapeModeAndBlur(config, options);//设置RequestOptions 关于 多重变换
        return options;
    }


    /**
     * 设置加载优先级
     *
     * @param config
     * @param options
     */
    private void setPriority(SingleConfig config, RequestOptions options) {
        switch (config.getPriority()) {
            case PriorityMode.PRIORITY_LOW:
                options.priority(Priority.LOW);
                break;
            case PriorityMode.PRIORITY_NORMAL:
                options.priority(Priority.NORMAL);
                break;
            case PriorityMode.PRIORITY_HIGH:
                options.priority(Priority.HIGH);
                break;
            case PriorityMode.PRIORITY_IMMEDIATE:
                options.priority(Priority.IMMEDIATE);
                break;
            default:
                options.priority(Priority.IMMEDIATE);
                break;
        }
    }

    /**
     * 设置加载进入动画
     *
     * @param config
     * @param request
     */
    private void setAnimator(SingleConfig config, RequestBuilder request) {
        if (config.getAnimationType() == AnimationMode.ANIMATIONID) {
            GenericTransitionOptions genericTransitionOptions = GenericTransitionOptions.with(config.getAnimationId());
            request.transition(genericTransitionOptions);
        } else if (config.getAnimationType() == AnimationMode.ANIMATOR) {
            GenericTransitionOptions genericTransitionOptions = GenericTransitionOptions.with(config.getAnimator());
            request.transition(genericTransitionOptions);
        } else if (config.getAnimationType() == AnimationMode.ANIMATION) {
            GenericTransitionOptions genericTransitionOptions = GenericTransitionOptions.with(new ViewAnimationFactory(config.getAnimation()));
            request.transition(genericTransitionOptions);
        } else {//设置默认的交叉淡入动画
            request.transition(DrawableTransitionOptions.withCrossFade());
        }
    }

    @Nullable
    private RequestBuilder getRequestBuilder(SingleConfig config) {

        RequestManager requestManager = Glide.with(config.getContext());
        RequestBuilder request = null;
        if (config.isAsBitmap()) {
            request = requestManager.asBitmap();

        } else if (config.isGif()) {
            request = requestManager.asGif();
        } else {
            request = requestManager.asDrawable();
        }
        if (!TextUtils.isEmpty(config.getUrl())) {
            request.load(ImageUtil.appendUrl(config.getUrl()));
            Log.e("TAG", "getUrl : " + config.getUrl());
        } else if (!TextUtils.isEmpty(config.getFilePath())) {
            request.load(ImageUtil.appendUrl(config.getFilePath()));
            Log.e("TAG", "getFilePath : " + config.getFilePath());
        } else if (!TextUtils.isEmpty(config.getContentProvider())) {
            request.load(Uri.parse(config.getContentProvider()));
            Log.e("TAG", "getContentProvider : " + config.getContentProvider());
        } else if (config.getResId() > 0) {
            request.load(config.getResId());
            Log.e("TAG", "getResId : " + config.getResId());
        } else if (config.getFile() != null) {
            request.load(config.getFile());
            Log.e("TAG", "getFile : " + config.getFile());
        } else if (!TextUtils.isEmpty(config.getAssertspath())) {
            request.load(config.getAssertspath());
            Log.e("TAG", "getAssertspath : " + config.getAssertspath());
        } else if (!TextUtils.isEmpty(config.getRawPath())) {
            request.load(config.getRawPath());
            Log.e("TAG", "getRawPath : " + config.getRawPath());
        }
        return request;
    }

    /**
     * 设置图片滤镜和形状
     *
     * @param config
     * @param options
     */
    private void setShapeModeAndBlur(SingleConfig config, RequestOptions options) {

        int count = 0;

        Transformation[] transformation = new Transformation[statisticsCount(config)];

        if (config.isNeedBlur()) {
            transformation[count] = new BlurTransformation(config.getBlurRadius());
            count++;
        }

        if (config.isNeedBrightness()) {
            transformation[count] = new BrightnessFilterTransformation(config.getBrightnessLeve()); //亮度
            count++;
        }

        if (config.isNeedGrayscale()) {
            transformation[count] = new GrayscaleTransformation(); //黑白效果
            count++;
        }

        if (config.isNeedFilteColor()) {
            transformation[count] = new ColorFilterTransformation(config.getFilteColor());
            count++;
        }

        if (config.isNeedSwirl()) {
            transformation[count] = new SwirlFilterTransformation(0.5f, 1.0f, new PointF(0.5f, 0.5f)); //漩涡
            count++;
        }

        if (config.isNeedToon()) {
            transformation[count] = new ToonFilterTransformation(); //油画
            count++;
        }

        if (config.isNeedSepia()) {
            transformation[count] = new SepiaFilterTransformation(); //墨画
            count++;
        }

        if (config.isNeedContrast()) {
            transformation[count] = new ContrastFilterTransformation(config.getContrastLevel()); //锐化
            count++;
        }

        if (config.isNeedInvert()) {
            transformation[count] = new InvertFilterTransformation(); //胶片
            count++;
        }

        if (config.isNeedPixelation()) {
            transformation[count] = new PixelationFilterTransformation(config.getPixelationLevel()); //马赛克
            count++;
        }

        if (config.isNeedSketch()) {
            transformation[count] = new SketchFilterTransformation(); //素描
            count++;
        }

        if (config.isNeedVignette()) {
            transformation[count] = new VignetteFilterTransformation(new PointF(0.5f, 0.5f),
                    new float[]{0.0f, 0.0f, 0.0f}, 0f, 0.75f);//晕映
            count++;
        }

        switch (config.getShapeMode()) {
            case ShapeMode.RECT:

                break;
            case ShapeMode.RECT_ROUND:
                transformation[count] = new RoundedCornersTransformation
                        (config.getRectRoundRadius(), 0, RoundedCornersTransformation.CornerType.ALL);
                count++;
                break;
            case ShapeMode.OVAL://@deprecated Use {@link RequestOptions#circleCrop()}.
//                transformation[count] = new CropCircleTransformation();
//                count++;
                options = options.circleCrop();
                break;

            case ShapeMode.SQUARE:
                transformation[count] = new CropSquareTransformation();
                count++;
                break;
        }

        if (transformation.length != 0) {
            options.transforms(transformation);
        }
    }
    ...
 }
 ```
