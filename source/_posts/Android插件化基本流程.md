---
title: Android插件化基本流程
photos:
-   /img/pictures/picture18.jpg 
date: 2019-05-08 16:40:38
tags:
categories:
---

> 插件化主要功能
>* 实现宿主程序调用插件中类方法的功能
>* 实现宿主程序获取插件中资源文件的功能
>* 实现宿主程序渲染插件中layout布局的功能

**该篇暂未涉及到四大组件**

<!--more-->

## 加载插件中的类文件

> 通多DexClassLoader加载器加载产检中的dex文件，相关文档可参考 [Android解析ClassLoader](http://liuwangshu.cn/application/classloader/2-android-classloader.html)

```java
//加载apk，生成对应的ClassLoader
DexClassLoader bundleDexClassLoader = new DexClassLoader(
f.getAbsolutePath(), dexDir.getAbsolutePath(), null,
 context.getClassLoader().getParent());
```

## 加载插件中的资源文件

> Android中是通过Resources、AssetManager来获取资源文件。为了避免插件和宿主中资源id冲突的问题，加载插件中资源时会创建新的Resources、AssetManage对象，其中AssetManage需要通过反射机制添加插件的资源路径

[参考文档插件化-资源加载 ](https://www.jianshu.com/p/96d5b83ca26c)

```java
AssetManager assetManager = AssetManager.class.newInstance();
try {
   AssetManager.class.getDeclaredMethod("addAssetPath", String.class).invoke(
         assetManager, apkPath);
} catch (Throwable th) {
   System.out.println("debug:createAssetManager :"+th.getMessage());
   th.printStackTrace();
}
return assetManager;

```

### 宿主环境渲染插件中的布局文件

> 上面已经实现了加载插件中资源和类文件的功能， 获取到插件的布局后通过LayoutInflater来渲染布局。需要注意的是，获取LayoutInflater实例时需要对Context对象进行封装，重写Context 的getResources()、getAssets()、以及getClssLoader()方法，返回对应插件的相关信息。

>同时需要重写getSystemService()实例化方法，通过cloneInContext()方法使设置的Context生效

参考 [360Replugin项目](https://github.com/Qihoo360/RePlugin/wiki)

相关分析文档 [Replugin- 插件的安装和加载原理](https://blog.csdn.net/yulong0809/article/details/78428247)

以及Replugin项目中的 com.qihoo360.loader2.PluginContext 类文件

```java
public class PluginWrapper extends ContextWrapper {
    private Resources mResources;
    private LayoutInflater mInflater;
    private ClassLoader mClassLoader;
    private String TAG=PluginWrapper.class.getSimpleName();
 
    public PluginWrapper(Context base,Resources resources,ClassLoader classLoader) {
        super(base);
        this.mResources = resources;
        this.mClassLoader=classLoader;
    }
 
 
 
    @Override
    public Object getSystemService(String name) {
        if (LAYOUT_INFLATER_SERVICE.equals(name)) {
            if (mInflater == null) {
                LayoutInflater inflater = (LayoutInflater) super.getSystemService(name);
//                // 新建一个，设置其工厂
                mInflater = inflater.cloneInContext(this);
            }
            return mInflater;
        }
        return super.getSystemService(name);
 
    }
 
    @Override
    public AssetManager getAssets() {
        if (mResources != null) {
            return mResources.getAssets();
        }
        return super.getAssets();
    }
 
 
    @Override
    public Resources getResources() {
        if (mResources != null) {
            return mResources;
        }
 
        return super.getResources();
    }
 
 
    /**
     *  重写getClassLoader()方法，
     *  布局渲染时加载插件包里面的class文件
     *  避免自定义布局 ClassNotFoundException 问题
     * @return 插件包的classloader
     */
    @Override
    public ClassLoader getClassLoader() {
        if (mClassLoader!=null){
            return mClassLoader;
        }
        Log.d(TAG,"getClassLoader()");
        return super.getClassLoader();
    }
 
 
}


```