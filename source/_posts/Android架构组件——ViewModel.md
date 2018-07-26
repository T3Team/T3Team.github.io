---
title: Android架构组件——ViewModel
date: 2018-07-02 10:16:01
author: 
tags:
categories: android技术文档
---
### 概述
Android 官方架构组件在2017年11月份Android官方架构组件正式版发布, 并且 Google 也在 Support Library v26.1.0 以后的版本中内嵌了 Android 官方架构组件中的生命周期组件.
>ViewModel是UI相关数据管理类。但不是数据持有类

最为重要的就是ViewModel具有下面的生命周期：

![Rvm_lf](/images/vm_lf.png)

上图是用Activity作为例子，左侧表示Activity的生命周期状态，右侧绿色部分表示ViewModel的生命周期范围。当屏幕旋转的时候，Activity会被recreate，Activity会经过几个生命周期方法，但是这个时候ViewModel还是之前的对象，并没有被重新创建，只有当Activity的finish()方法被调用时，ViewModel.onCleared()方法会被调用，对象才会被销毁。这张图很好的描述了是当Activity被recreate时，ViewModel的生命周期。
注意：**在ViewModel中不要持有Activity的引用**。从上面的图我们看到，当Activity被recreate时，ViewModel对象并没有被销毁，如果Model持有Activity的引用时就可能会导致内存泄漏。那如果你要使用到Context对象，建议使用ViewModel的子类AndroidViewModel或者你自己传入application的上下文。

**正因为ViewModel有如此的生命周期，所以ViewModel在MVVM可以作为数据管理，是连接View和Model重要组件，ViewModel的核心作用如下图所示：**

![vm_fuc](/images/vm_fuc.png)

**ViewModel是怎么创建的？**
ViewModel的基本使用方法，我们在获取ViewModel的时候绝对不能直接使用new关键字去创建，需要使用 ViewModelProviders 去使用系统提供的反射方法去创建我们想要的ViewModel，下面是官方架构组件android.arch.lifecycle包下面的ViewModelProviders工具类用来获取ViewModel:

``` 
public class ViewModelProviders {

    /**
     * 通过Activity获取可用的Application
     * 或者检测Activity是否可用
     * @param activity
     * @return
     */
    private static Application checkApplication(Activity activity) {
        Application application = activity.getApplication();
        if (application == null) {
            throw new IllegalStateException("Your activity/fragment is not yet attached to "
                    + "Application. You can't request ViewModel before onCreate call.");
        }
        return application;
    }

    /**
     * 通过Fragment获取Activity
     * 或者检测Fragment是否可用
     * @param fragment
     * @return
     */
    private static Activity checkActivity(Fragment fragment) {
        Activity activity = fragment.getActivity();
        if (activity == null) {
            throw new IllegalStateException("Can't create ViewModelProvider for detached fragment");
        }
        return activity;
    }

    /**
     * 通过Fragment获得ViewModelProvider
     * @param fragment
     * @return
     */
    @NonNull
    @MainThread
    public static ViewModelProvider of(@NonNull Fragment fragment, @Nullable Factory factory) {
        Application application = checkApplication(checkActivity(fragment));
        if (factory == null) {
        	/**获取默认的单例AndroidViewModelFactory，它内部是通过反射来创建具体的ViewModel*/
            factory = ViewModelProvider.AndroidViewModelFactory.getInstance(application);
        }
        /***
         *   利用HolderFragment来关联生命周期并使用HolderFragment中的ViewModelStore的HashMap存储ViewModel
         *   AndroidViewModelFactory创建ViewModel
         */
        return new ViewModelProvider(ViewModelStores.of(fragment), factory);
    }

    /**
     * 通过FragmentActivity获得ViewModelProvider
     * @param activity
     * @return
     */
    @NonNull
    @MainThread
    public static ViewModelProvider of(@NonNull FragmentActivity activity,
            @Nullable Factory factory) {
        Application application = checkApplication(activity);
        /**获取默认的单例AndroidViewModelFactory，它内部是通过反射来创建具体的ViewModel*/
        if (factory == null) {
            factory = ViewModelProvider.AndroidViewModelFactory.getInstance(application);
        }
        /***
         *   利用HolderFragment来关联生命周期并使用HolderFragment中的ViewModelStore的HashMap存储ViewModel
         *   AndroidViewModelFactory创建ViewModel
         */
        return new ViewModelProvider(ViewModelStores.of(activity), factory);
    }
}

```
**创建使用ViewModel:** 
```
//传入对应的上下文 即：数据retain的宿主
NetDemoViewModel  netDemoViewModel = ViewModelProviders.of(fragment/activity).get(NetDemoViewModel.class);

```
ViewModel 的存在是依赖 Activity 或者 Fragment的，不管你在什么地方获取ViewModel ，只要你用的是相同的Activity 或者 Fragment，那么获取到的ViewModel将是同一个 (前提是key值是一样的)，所以ViewModel 有数据共享的作用。


** 那ViewModel是怎么创建的？**
看上面的获取viewmodel的对象的链式调用的方法可以理解成 分为两步
```
/*****第一步:根据Activity或者Fragment获得ViewModelProvider****/
ViewModelProviders viewModelProvider = ViewModelProviders.of(fragment/activity);
/*****第二步:使用ViewModelProvider反射创建需要的ViewModel****/
NetDemoViewModel  netDemoViewModel = viewModelProvider.get(NetDemoViewModel.class);

```
第一步获得的源代码：

```
    /**
     * Creates a {@link ViewModelProvider}, which retains ViewModels while a scope of given
     * {@code fragment} is alive. More detailed explanation is in {@link ViewModel}.
     * <p>
     * It uses the given {@link Factory} to instantiate new ViewModels.
     *
     * @param fragment a fragment, in whose scope ViewModels should be retained
     * @param factory  a {@code Factory} to instantiate new ViewModels
     * @return a ViewModelProvider instance
     */
    @NonNull
    @MainThread
    public static ViewModelProvider of(@NonNull Fragment fragment, @Nullable Factory factory) {
        Application application = checkApplication(checkActivity(fragment));
        if (factory == null) {
            /**********获得AndroidViewModelFactory ( 内部是单例的 )*******/
            factory = ViewModelProvider.AndroidViewModelFactory.getInstance(application);
        }
         /*****创建一个ViewModelProvider( 传入的两个参数是重点 )*****/
        return new ViewModelProvider(ViewModelStores.of(fragment), factory);
    }

```
上面的两步，获得AndroidViewModelFactory ，AndroidViewModelFactory其实是ViewModelProvider的静态内部类，看调用方式就知道是一个单例的，就是应用里面只有一个单例的 AndroidViewModelFactory存在，看源码
```
 public static class AndroidViewModelFactory extends ViewModelProvider.NewInstanceFactory {

        private static AndroidViewModelFactory sInstance;

        /**
         * Retrieve a singleton instance of AndroidViewModelFactory.
         * 获得AndroidViewModelFactory 单例
         * @param application an application to pass in {@link AndroidViewModel}
         * @return A valid {@link AndroidViewModelFactory}
         */
        @NonNull
        public static AndroidViewModelFactory getInstance(@NonNull Application application) {
            if (sInstance == null) {
                sInstance = new AndroidViewModelFactory(application);
            }
            return sInstance;
        }

        private Application mApplication;

        /**
         * Creates a {@code AndroidViewModelFactory}
         *
         * @param application an application to pass in {@link AndroidViewModel}
         */
        public AndroidViewModelFactory(@NonNull Application application) {
            mApplication = application;
        }
       /******其实这里就是创建ViewModel的关键地方，根据给出的Class反射创建需要的ViewModel*******/
        @NonNull
        @Override
        public <T extends ViewModel> T create(@NonNull Class<T> modelClass) {
            if (AndroidViewModel.class.isAssignableFrom(modelClass)) {
                //noinspection TryWithIdenticalCatches
                try {
                    return modelClass.getConstructor(Application.class).newInstance(mApplication);
                } catch (NoSuchMethodException e) {
                    throw new RuntimeException("Cannot create an instance of " + modelClass, e);
                } catch (IllegalAccessException e) {
                    throw new RuntimeException("Cannot create an instance of " + modelClass, e);
                } catch (InstantiationException e) {
                    throw new RuntimeException("Cannot create an instance of " + modelClass, e);
                } catch (InvocationTargetException e) {
                    throw new RuntimeException("Cannot create an instance of " + modelClass, e);
                }
            }
            return super.create(modelClass);
        }
    }

```
看到这里,全局的AndroidViewModelFactory工具类，作用其实就是反射创建我们想要的类ViewModel。
获得到的单例AndroidViewModelFactory是创建ViewModelProvider的第二个参数。
第一个参数是这样的： ViewModelStores.of(activity) 
看源码
```
public class ViewModelStores {

    private ViewModelStores() {
    }

    /**
     * Returns the {@link ViewModelStore} of the given activity.
     *
     * @param activity an activity whose {@code ViewModelStore} is requested
     * @return a {@code ViewModelStore}
     */
    @NonNull
    @MainThread
    public static ViewModelStore of(@NonNull FragmentActivity activity) {
    	//如果你的Activity实现了ViewModelStoreOwner接口具备了提供
        //ViewModelStore 的功能就直接获取返回，通常我们的Activity都不会去实现这个功能
        if (activity instanceof ViewModelStoreOwner) {
            return ((ViewModelStoreOwner) activity).getViewModelStore();
        }
        //系统为你的Activity添加一个具有提供ViewModelStore 的holderFragment
        return holderFragmentFor(activity).getViewModelStore();
    }
}

```
其实理解ViewModelStore就可以解释ViewModel的存储，理解了holderFragmentFor(activity).getViewModelStore() 就可解释ViewModel为什么可以在Activity配置发生变化的情况下人不销毁。
以上源码分析可以明白：
+ AndroidViewModelFactory在正常情况下是全局单例只有一个，只是一个反射创建对象的工具类。 
+ ViewModelProvider是每次获取创建ViewModel的时候都会创建一个新的。 
+ ViewModelStore是每一个Activity或者Fragment都有一个。

第二部分:
```
@MainThread
public <T extends ViewModel> T get(@NonNull Class<T> modelClass) {
    String canonicalName = modelClass.getCanonicalName();
    if (canonicalName == null) {
        throw new IllegalArgumentException("Local and anonymous classes can not be ViewModels");
    }
    return get(DEFAULT_KEY + ":" + canonicalName, modelClass);
}
```
用 DEFAULT_KEY 和 类名组成一个key值去获取;

```
   /**
     * Returns an existing ViewModel or creates a new one in the scope (usually, a fragment or
     * an activity), associated with this {@code ViewModelProvider}.
     * <p>
     * The created ViewModel is associated with the given scope and will be retained
     * as long as the scope is alive (e.g. if it is an activity, until it is
     * finished or process is killed).
     *
     * @param key        The key to use to identify the ViewModel.
     * @param modelClass The class of the ViewModel to create an instance of it if it is not
     *                   present.
     * @param <T>        The type parameter for the ViewModel.
     * @return A ViewModel that is an instance of the given type {@code T}.
     */
    @NonNull
    @MainThread
    public <T extends ViewModel> T get(@NonNull String key, @NonNull Class<T> modelClass) {
        ViewModel viewModel = mViewModelStore.get(key);

        if (modelClass.isInstance(viewModel)) {
            //noinspection unchecked
            return (T) viewModel;
        } else {
            //noinspection StatementWithEmptyBody
            if (viewModel != null) {
                // TODO: log a warning.
            }
        }

        viewModel = mFactory.create(modelClass);
        mViewModelStore.put(key, viewModel);
        //noinspection unchecked
        return (T) viewModel;
    }
```
代码很简单，流程如下: 
+ 先从mViewModelStore中使用key去获取ViewModel, mViewModelStore中是使用HashMap去存储一个Activity或者Fragment的ViewModel的。如果获取到就返回。 
+ 没获取到就使用单例mFactory的create方法反射创建ViewModel,create方法的代码在上面贴出来了。 
+ 使用Key存入mViewModelStore 并返回。

也就是创建一个ViewModelProvider，使用ViewModelProvider内部的全局单例AndroidViewModelFactory来反射创建 ViewModel,并把创建的ViewModel存入传入的ViewModelStore中.

**ViewModel是怎么存储**

```
public class ViewModelStore {

    private final HashMap<String, ViewModel> mMap = new HashMap<>();

    final void put(String key, ViewModel viewModel) {
        ViewModel oldViewModel = mMap.put(key, viewModel);
        if (oldViewModel != null) {
            oldViewModel.onCleared();
        }
    }

    final ViewModel get(String key) {
        return mMap.get(key);
    }

    /**
     *  Clears internal storage and notifies ViewModels that they are no longer used.
     */
    public final void clear() {
        for (ViewModel vm : mMap.values()) {
            vm.onCleared();
        }
        mMap.clear();
    }
}
```
就是一个 HashMap用存储ViewModel。提供get,put,clear三个方法。 ViewModelStore是每一个Activity或者Fragment都有一个的，当Activity或者Fragment销毁的时候就会调用clear方法了。

通过上面看源码:

ViewModelStore被谁创建，被谁持有？
被HolderFragment创建和持有！

HolderFragment跟我们的Activity或者Fragment有什么关系？
当我们要给Activity或者Fragment创建ViewModel的时候，系统就会为Activity或者Fragment添加一个HolderFragment，HolderFragment中会创建持有一个ViewModelStore。

HolderFragment怎么创建怎么被添加？
new ViewModelProvider(ViewModelStores.of(activity), factory);//上面有分析如何创建
为什么都添加一个HolderFragment？
```
//HolderFragment架造
public HolderFragment() {
     setRetainInstance(true);
}
```
setRetainInstance(boolean) 是Fragment中的一个方法。将这个方法设置为true就可以使当前Fragment在Activity重建时存活下来, 如果不设置或者设置为 false, 当前 Fragment 会在 Activity 重建时同样发生重建, 以至于被新建的对象所替代。 
在setRetainInstance(boolean)为true的 Fragment （就是HolderFragment）中放一个专门用于存储ViewModel的Map, 这样Map中所有的ViewModel都会幸免于Activity的配置改变导致的重建，让需要创建ViewModel的Activity, Fragment都绑定一个这样的Fragment（就是HolderFragment）, 将ViewModel存放到这个 Fragment 的 Map 中, ViewModel 组件就这样实现了。
**总结**
1.ViewModel 以键值对的形式存在Activity或者Fragment的HolderFragment的 
ViewModelStore的HashMap中。
2.一个Activity或者Fragment可以有很多个ViewModel。
3.一个Activity或者Fragment只会有一个HolderFragment。
4.Activity或者Fragment的HolderFragment会保存在全局单例的HolderFragmentManager的HashMap中，在Activity或者Fragment销毁的时候会移除HashMap中对应的value。
5.因为ViewModel是以Activity或者Fragment为存在基础，所以ViewModel可以在当前Activity和Fragment中实现数据共享，前提是传入相同的key值。
所以ViewModel 主要就两个功能
+ 第一个功能可以使 ViewModel 以及 ViewModel 中的数据在屏幕旋转或配置更改引起的 Activity 重建时存活下来, 重建后数据可继续使用。
+ 第二个功能可以帮助开发者轻易实现 Fragment 与 Fragment 之间, Activity 与 Fragment 之间的通讯以及共享数据


