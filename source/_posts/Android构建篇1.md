---
title: Android 构建篇(1)：Gradle 基础知识
date: 2017-08-02
tags: 
categories: Android
---

本文阐述 Android 使用 Gradle 构建的基础知识，包括
- Gradle
- Gradle 与 Groovy
- Gradle 与 Android Studio
- Gradle 版本配置
- Gradle 与 Gradle Wrapper
- Gradle 与仓库

<!-- more -->

## 一、Gradle 是什么
[Gradle](https://gradle.org/) 是一个自动化构建工具，独立于 Android、Android Studio，具备依赖管理、编译、运行、签名、打包等功能


## 二、Gradle 与 Groovy 的关系？
Gradle 以 Groovy 语言为基础，类似 Ant 是基于 XML 的
[Apache Groovy](http://groovy-lang.org/)提到
>A multi-faceted language for the Java platform
>Apache Groovy is a powerful, optionally typed and dynamic language, with static-typing and static compilation capabilities, for the Java platform aimed at improving developer productivity thanks to a concise, familiar and easy to learn syntax. It integrates smoothly with any Java program, and immediately delivers to your application powerful features, including scripting capabilities, Domain-Specific Language authoring, runtime and compile-time meta-programming and functional programming.

### 1. Groovy 是一门动态语言，并支持静态类型及静态编译，语法类似Java，并具有 Python Ruby 等语言特性
### 2. Groovy 运行于 JVM 中
每次运行 Groovy 脚本，都会先将其编译为 Java 字节码，然后通过 JVM 来执行
### 3. Groovy 是一门 领域特定语言 Domain-Specific Language
引用 [DSL（Domain-Specific Language，领域特定语言）是什么](http://yangbinfx.iteye.com/blog/1917334)
>DSL 就是针对某个领域所设计出来的一个特定的语言。因为有了领域的限制，要解决的问题就被划定了范围，所以语言不需要复杂，就可以具有精确的表达能力。

### 4. Groovy 支持 元编程(meta-programming) 及 函数式编程(functional programming)

[重新认识 Android Studio 和 Gradle，这些都是我们应该知道的](https://zhuanlan.zhihu.com/p/22990436)提到
>我们在 build.gradle 里边不是简单的配置，而是直接的逻辑开发


## 三、Gradle 与 Android Studio 的关系？
上文提到，Gradle 本质上与 Android Studio 无关
为了支持 Gradle 能在 Android Studio 上使用，Google 推出了 Android Gradle Plugin


## 四、Gradle 有多处版本配置？版本信息不一致？
### 1. 配置 Gradle 版本
我们借助 Gradle Wrapper 来使用 Gradle
Gradle Wrapper 配置文件: $rootProject.projectDir/gradle-wrapper.properties
```
...
distributionUrl=https\://services.gradle.org/distributions/gradle-2.14.1-all.zip
```
### 2. 配置 Android Gradle Plugin 版本
上文提到，Android Gradle Plugin 由 Google 提供，版本号由 Google 指定，与 Gradle 官方无关
$rootProject.projectDir/build.gradle
```
buildscript {
    ...
    dependencies {
        classpath 'com.android.tools.build:gradle:2.2.2'
    }
}
```
### 3. Gradle 与 Android Gradle Plugin 版本必须一一对应
[Android Plugin for Gradle Release Notes - Update Gradle](https://developer.android.com/studio/releases/gradle-plugin.html#updating-gradle) 提供对应表格


## 五、Gradle 与 Gradle Wrapper 的关系？
在 Android Studio 新建一个 Android Project，默认会直接安装 [Gradle Wrapper](https://docs.gradle.org/current/userguide/gradle_wrapper.html)(而非 Gradle)，也就是 Gradle 的包装器

### 1. 为什么使用 gradlew 命令而非 gradle？
假设这么一种情况，一个 project 有很多个 module，每个 module 的 Gradle 版本配置不一致，以前的 module Gradle 版本较旧，新建的 module Gradle 版本较新
若使用 gradle 命令，我就必须要将每个版本的 gradle 都配置到环境变量中
若使用 gradlew 命令，就避免了这种麻烦

### 2. Gradle Wrapper
gradlew 为 Gradle Wrapper 的缩写，Gradle Wrapper 是 Gradle 的封装，Gradle Wrapper 用于给 Gradle 使用者带来好处
一个 Gradle Wrapper 对应一个特定版本的 Gradle，当用户第一次执行 gradlew 命令时，Gradle Wrapper 会自动下载、安装对应版本的 Gradle，这就带来两个好处: 
(1) 用户不必自己下载、安装、配置Gradle
(2) 构建时保证编译环境统一

### 3. 使用 gradlew
$rootProject.projectDir/gradlew
示例: 查看 Gradle 版本信息
```
$ ./gradlew -v

------------------------------------------------------------
Gradle 2.14.1
------------------------------------------------------------

Build time:   2016-07-18 06:38:37 UTC
Revision:     d9e2113d9fb05a5caabba61798bdb8dfdca83719

Groovy:       2.4.4
Ant:          Apache Ant(TM) version 1.9.6 compiled on June 29 2015
JVM:          1.8.0_112 (Oracle Corporation 25.112-b16)
OS:           Mac OS X 10.12.1 x86_64
```

### 4. 常用 gradlew 命令
(1) gradlew -v
查看 Gradle 版本信息
(2) gradlew clean
清空 build 文件夹
(3) gradlew build
编译打包，同时包括 debug/release
(4) gradlew assembleDebug/gradlew assembleRelease
编译打包，debug 或 release
注: assemble 是和 Build Variants 一起结合使用，而 Build Variants == Build Type + Product Flavor，而Product Flavor 用于多渠道打包
(5) gradlew installRelease
打 Release 包并安装
(6) gradlew uninstallRelease
卸载Release包


## 六、Android Studio 是从哪里下载资源的?
Gradle 会在 repository(仓库) 里找资源文件，仓库本质上是文件的集合，通过 groupId、$artifactId、$version整理分类

### 1. jcenter 与 mavenCentral
Android Studio Gradle 支持两个 Maven 中央库: jcenter 与 mavenCentral
二者托管地址完全不同，内容由不同提供者提供的，并且之间并没有任何关联
也就是可能存在一种情况，在 jcenter 能找到的资源，在 mavenCentral 找不到

### 2. 声明依赖
依赖Android Gradle Plugin 2.2.2版本，并从 jcenter 下载
$rootProject.projectDir/build.gradle
```
buildscript {
    repositories {
        jcenter()
    }
    dependencies {
        classpath 'com.android.tools.build:gradle:2.2.2'

        // NOTE: Do not place your application config here; they belong
        // in the individual module build.gradle files
    }
}
```
配置依赖需指定 aar/jar 标识: $groupId:$artifactId:$version
(1) groupId: com.android.tools.build
(2) artifactId: gradle
(3) version: 2.2.2

### 3. 下载资源
清空 gradle 缓存，并构建
```
$rm -rf ~/.gradle
#./gradlew build
Downloading https://services.gradle.org/distributions/gradle-2.14.1-all.zip
...........................................................................
Unzipping /Users/yutinglin/.gradle/wrapper/dists/gradle-2.14.1-all/8bnwg5hd3w55iofp58khbp6yv/gradle-2.14.1-all.zip to /Users/yutinglin/.gradle/wrapper/dists/gradle-2.14.1-all/8bnwg5hd3w55iofp58khbp6yv
Set executable permissions for: /Users/yutinglin/.gradle/wrapper/dists/gradle-2.14.1-all/8bnwg5hd3w55iofp58khbp6yv/gradle-2.14.1/bin/gradle
To honour the JVM settings for this build a new JVM will be forked. Please consider using the daemon: https://docs.gradle.org/2.14.1/userguide/gradle_daemon.html.
Generating JAR file 'gradle-api-2.14.1.jar'
Download https://jcenter.bintray.com/com/android/tools/build/gradle/2.2.2/gradle-2.2.2.pom
...

```
#### 什么是 Maven 仓库
Maven 仓库集中存放 Maven Package
Maven 仓库可以在本地，也可以在远程服务器上；可以是私有的，也可以是公开的
#### 什么是 Maven Package
Maven Package 格式由 POM(Project Object Model) 定义
#### 什么是 POM(Project Object Model) 文件
本质上是个 XML 文件，定义 Maven Package 格式
所有 POM 文件都需要定义 groupId artifactId version
在 Maven 仓库中的标识: $groupId:$artifactId:$version
示例: [gradle-2.2.2.pom](https://jcenter.bintray.com/com/android/tools/build/gradle/2.2.2/gradle-2.2.2.pom)
```
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">
  <modelVersion>4.0.0</modelVersion>
  <groupId>com.android.tools.build</groupId>
  <artifactId>gradle</artifactId>
  <version>2.2.2</version>
  <dependencies>
    <dependency>
      <groupId>com.android.tools.build</groupId>
      <artifactId>gradle-core</artifactId>
      <version>2.2.2</version>
      <scope>runtime</scope>
    </dependency>
  </dependencies>
  <description>Gradle plug-in to build Android applications.</description>
  <url>http://tools.android.com/</url>
  <name>com.android.tools.build.gradle</name>
  <licenses>
    <license>
      <name>The Apache Software License, Version 2.0</name>
      <url>http://www.apache.org/licenses/LICENSE-2.0.txt</url>
      <distribution>repo</distribution>
    </license>
  </licenses>
  <developers>
    <developer>
      <name>The Android Open Source Project</name>
    </developer>
  </developers>
  <scm>
    <connection>git://android.googlesource.com/platform/tools/base.git</connection>
    <url>https://android.googlesource.com/platform/tools/base</url>
  </scm>
</project>
```

### 4. 发布
关于发布资源到 jcenter 与 mavenCentral 远程库、Maven 本地仓库，搭建 Maven 私有仓库，参考以下文章
[如何通过 Android Studio 发布 library 到 jCenter 和 Maven Central](http://www.jianshu.com/p/3c63ae866e52)
[拥抱 Android Studio 之四: Maven 仓库使用与私有仓库搭建](http://blog.bugtags.com/2016/01/27/embrace-android-studio-maven-deploy/)


## 七、Project 与 Task

Gradle 脚本执行时，本质上会使用某些特定类型的对象
![Project 与 Task](https://raw.githubusercontent.com/yuting-lin/hexo.github.io/master/source/_posts/Android%E6%9E%84%E5%BB%BA%E7%AF%87/Project%20%E4%B8%8E%20Task.png)

### 1. Project
每个 build.gradle 都对应一个 [Project](https://docs.gradle.org/current/dsl/org.gradle.api.Project.html) 对象
每个构建由一个或多个 [Project](https://docs.gradle.org/current/dsl/org.gradle.api.Project.html) 构成，一个 Project 可以代表构建一个 jar，也可以代表部署一个 jar

### 2. 生命周期
(1) 创建 [Settings](https://docs.gradle.org/4.0/dsl/org.gradle.api.initialization.Settings.html) 对象
(2) 执行 settings.gradle
计算出 project & modules 的依赖关系(树状图)
(3) 根据 project & modules 依赖关系，执行每个 build.gradle
计算出所有 tasks 的依赖关系(有向图)
(4) 根据 tasks 的依赖关系，执行 task

### 3. Task
(1) 每个 [Project](https://docs.gradle.org/current/dsl/org.gradle.api.Project.html) 由一个或多个 [Task](https://docs.gradle.org/current/dsl/org.gradle.api.Task.html) 构成
(2) 一个 [Task](https://docs.gradle.org/current/dsl/org.gradle.api.Task.html) 代表更细化的任务，可以是编译 class、生成压缩文件等
(3) 每个 [Task](https://docs.gradle.org/current/dsl/org.gradle.api.Task.html) 属于一个 [Project](https://docs.gradle.org/4.0/dsl/org.gradle.api.Project.html) 对象
更多信息参考 [More about Tasks](https://docs.gradle.org/4.0/userguide/more_about_tasks.html)

## 八、插件
引用 [Project 文档](https://docs.gradle.org/current/dsl/org.gradle.api.Project.html)
>Plugins can be used to modularise and reuse project configuration. Plugins can be applied using the [PluginAware.apply(java.util.Map)](https://docs.gradle.org/current/dsl/org.gradle.api.plugins.PluginAware.html#org.gradle.api.plugins.PluginAware:apply(java.util.Map)) method, or by using the [PluginDependenciesSpec](https://docs.gradle.org/current/dsl/org.gradle.plugin.use.PluginDependenciesSpec.html).

### 1. 插件的作用
插件封装了可重用的构建逻辑，可以适用于不同的项目和构建过程
Gradle 提供了很多官方插件，用于支持 Java、Groovy 等工程的构建和打包
引用[插件的作用是什么](https://dongchuan.gitbooks.io/gradle-user-guide-/gradle_plugins/what_plugins_do.html)
>应用插件到项目允许插件来扩展项目的能力。它可以做的事情，如：
 扩展摇篮模型（如:添加可配置新的DSL元素）
 按照惯例配置项目(如:添加新的任务或配置合理的默认值)
 应用特定的配置（如:增加组织库或执行标准）
>通过应用插件,而不是向项目构建脚本添加逻辑,我们可以收获很多好处.应用插件:
 促进重用和减少维护在多个项目类似的逻辑的开销
 允许更高程度的模块化，提高综合性和组织
 封装必要的逻辑，并允许构建脚本尽可能是声明性地

### 2. 插件打包方式
(1) 插件实现在 build.gradle，只有该构建脚本可见
(2) buildSrc module，只有该 project 可见
(3) 独立 module，需发布到托管平台上
### 3. 自定义插件
引用 [Writing Custom Plugins](https://docs.gradle.org/current/userguide/custom_plugins.html)
```
apply plugin: GreetingPlugin

class GreetingPlugin implements Plugin<Project> {
    void apply(Project project) {
        project.task('hello') {
            doLast {
                println "Hello from the GreetingPlugin"
            }
        }
    }
}
```


## 参考
[Gradle 文档](https://gradle.org/docs)
[The Gradle Wrapper](https://docs.gradle.org/current/userguide/gradle_wrapper.html)
[Apache Groovy](http://groovy-lang.org/)
[Android Plugin for Gradle Release Notes - Update Gradle](https://developer.android.com/studio/releases/gradle-plugin.html#updating-gradle)
[DSL（Domain-Specific Language，领域特定语言）是什么](http://yangbinfx.iteye.com/blog/1917334)
[Gradle User Guide](https://docs.gradle.org/4.0/userguide/userguide.html)
[Gradle User Guide 中文版](https://dongchuan.gitbooks.io/gradle-user-guide-/)
[Gradle Build Language Reference](https://docs.gradle.org/4.0/dsl/index.html)
[Gradle API Documentation - Project](https://docs.gradle.org/current/dsl/org.gradle.api.Project.html)
[More about Tasks](https://docs.gradle.org/4.0/userguide/more_about_tasks.html)
[重新认识 Android Studio 和 Gradle，这些都是我们应该知道的](https://zhuanlan.zhihu.com/p/22990436)
[Wrapper (gradlew)](https://www.zybuluo.com/xtccc/note/275168)
[给 ANDROID 初学者的 GRADLE 知识普及](http://stormzhang.com/android/2016/07/02/gradle-for-android-beginners/)
[如何通过 Android Studio 发布 library 到 jCenter 和 Maven Central](http://www.jianshu.com/p/3c63ae866e52)
[Writing Custom Plugins](https://docs.gradle.org/current/userguide/custom_plugins.html)
[拥抱 Android Studio 之四: Maven 仓库使用与私有仓库搭建](http://blog.bugtags.com/2016/01/27/embrace-android-studio-maven-deploy/)
[拥抱 Android Studio 之五：Gradle 插件开发](http://kvh.io/cn/embrace-android-studio-gradle-plugin.html)