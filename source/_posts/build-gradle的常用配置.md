---
title: build.gradle的常用配置
photos:
  - '/img/pictures/picture16.jpg'
date: 2018-03-26 16:27:53
tags: [gradle]
categories: [Android]
---
> 这篇主要记录下`build.gradle`文件的常用配置，例如修改打包命名和路径、统一管理多`moudle`的依赖库版本等。根据项目中具体用到的配置，这篇也会持续更新。[官方文档](https://developer.android.com/studio/build/index.html)

<!--more-->

## 修改apk名称

> 通过AS工具打包默认命名规则为`app-debug.apk`或者`app-release.apk`。实际使用中可能需要命名规则中包含更多信息，例如版本名称、打包时间、渠道信息等。这种效果可以在项目的`build.gradle`文件中添加配置信息来实现。

```gradle
//    修改打包后app命名开始
    def today = new Date().format('MMddHHmm')
    def name = new String(defaultConfig.applicationId)
            .substring(defaultConfig.applicationId.lastIndexOf(".")+1)
    android.applicationVariants.all { variant ->
        variant.outputs.each { output ->
            def file = output.outputFile
            output.outputFile = new File(file.parent, name + "_" + buildType.name + "_V" + defaultConfig.versionName + "_" + today + ".apk")
        }
    }
    //    修改打包后app命名结束
```
gradle3.0之后需要使用下面方法

```gradle
 //修改打包命名开始
    def today = new Date().format('MMddHHmm')
    def name = new String(defaultConfig.applicationId)
            .substring(defaultConfig.applicationId.lastIndexOf(".") + 1)
    android.applicationVariants.all { variant ->
        variant.outputs.all {
            outputFileName = name + "_" + buildType.name + "_V" + defaultConfig.versionName + "_" + today+".apk"
        }
    }
    //修改打包命名结束
```

| 属性 | column |
|:--------|--------:|
| output.outputFile.parent       |  生成apk的路径，这个是默认路径在项目/app/build/outpus/apk      |
|buildType.name|编译类型，就是debug或者release |
|defaultConfig.versionName|在defaultConfig标签下写的版本号|
|today|定义的打包时间字符串|

## 统一依赖库版本管理

> 如果项目中存在多个moudle的话，为了避免moudle的依赖库版本不统一导致重复依赖，最好统一配置依赖版本。下面提供两种配置方式。

### 在顶级`build.gradle`文件中配置依赖版本信息

1. 在顶级`build.gradle`文件中添加`ext`代码块

```gradle
// Top-level build file where you can add configuration options common to all sub-projects/modules.

buildscript {

    repositories {
        google()
        jcenter()
    }
    dependencies {
        classpath 'com.android.tools.build:gradle:3.0.1'

        // NOTE: Do not place your application dependencies here; they belong
        // in the individual module build.gradle files
    }
}

allprojects {
    repositories {
        google()
        jcenter()
    }
}

task clean(type: Delete) {
    delete rootProject.buildDir
}

ext {
    compileSdkVersion = 26
    minSdkVersion = 19
    targetSdkVersion = 26
    versionCode = 1
    versionName = '1.0'

    supportLibraryVersion='26.1.0'


}
```

2. 在各个`moudle`的`build.gradle`引用配置信息

```gradle
android {
  // Use the following syntax to access properties you defined at the project level:
  // rootProject.ext.property_name
  compileSdkVersion rootProject.ext.compileSdkVersion
  buildToolsVersion rootProject.ext.buildToolsVersion
  ...
}
...
dependencies {
    compile "com.android.support:appcompat-v7:${rootProject.ext.supportLibVersion}"
    ...
}

```

### 在`gradle.properties`文件中配置依赖版本信息

1. 在`gralde.properties`文件中添加依赖版本信息

```gradle
# When configured, Gradle will run in incubating parallel mode.
# This option should only be used with decoupled projects. More details, visit
# http://www.gradle.org/docs/current/userguide/multi_project_builds.html#sec:decoupled_projects
# org.gradle.parallel=true
TARGET_SDK_VERSION=21
COMPILE_SDK_VERSION=25
BUILD_TOOL_SVERSION=25.0.2
MIN_SDK_VERSION=14
SUPPORT_VERSION=25.4.0
```
2. 在各个`moudle`的`build.gradle`引用配置信息

> 在`gradle.properties`中配置的信息会作为字符串处理，如果参数需要数值类型的话需要在后面添加`as int`

```gradle
android {
  // Use the following syntax to access properties you defined at the project level:
  // rootProject.ext.property_name
  compileSdkVersion COMPILE_SDK_VERSION as int
  buildToolsVersion BUILD_TOOL_SVERSION
  ...
}
...
dependencies {
    compile "com.android.support:appcompat-v7:${SUPPORT_VERSION}"
    compile 'com.android.support:design:' + SUPPORT_VERSION
    ...
}

```

## 与应用代码共享自定义字段和资源


> 在构建时，Gradle 将生成 BuildConfig 类，以便应用代码可以检查与当前构建有关的信息。可以使用 buildConfigField() 函数，将自定义字段添加到 Gradle 构建配置文件的 BuildConfig 类中，然后在应用的运行时代码中访问这些值。同样，也可以使用 resValue() 添加应用资源值。[官方文档](https://developer.android.com/studio/build/gradle-tips.html)


```gradle
android {
  
  buildTypes {
    release {
      // These values are defined only for the release build, which
      // is typically used for full builds and continuous builds.
      buildConfigField("String", "BUILD_TIME", "\"${minutesSinceEpoch}\"")//在BuildConfild类中生成BUILD_TIME字段
      resValue("string", "build_time", "${minutesSinceEpoch}") // 在资源文件中生成build_time 字符串，目录为app/build/generated/res/resValues/[flavor-name]/[buildType]/values/generated.xml
      ...
    }
    debug {
      // Use static values for incremental builds to ensure that
      // resource files and BuildConfig aren't rebuilt with each run.
      // If they were dynamic, they would prevent certain benefits of
      // Instant Run as well as Gradle UP-TO-DATE checks.
      buildConfigField("String", "BUILD_TIME", "\"0\"")
      resValue("string", "build_time", "0")
    }
  }
}

```

在应用代码中使用

```
Log.i(TAG, BuildConfig.BUILD_TIME);
Log.i(TAG, getString(R.string.build_time));

```