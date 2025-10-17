---
title: 在国内如何更优雅的使用Gradle
published: 2025-01-20
tags: ['gradle','java','kotlin']
description: 'gradle 大势所趋'
category: '技术'
---

# 在国内如何更优雅的使用Gradle

## gradle wrapper下载缓慢

虽然gradle现在也有cdn了，但有时候难免遇到下载`gradle-8.12-all.zip`或者`gradle-8.12-bin.zip`下载不动的情况。这个时候一般都会打开代理软件去解决，但还有一个不用开代理的方式来解决这个问题修改`gradle/wrapper/gradle-wrapper.properties`这个文件中`distributionUrl`的路解决如下。保留`gradle-8.12-all.zip`将前面的url替换为`https://mirrors.cloud.tencent.com/gradle`。

```properties
distributionUrl=https://mirrors.cloud.tencent.com/gradle/gradle-8.12-all.zip
```

## gradle依赖下载缓慢

一般来说我们都会配置阿里或者腾讯的maven镜像仓库，但是gradle的某些依赖可能不会走我们配置的镜像，这个时候就可以选择在`settings.gradle.kts`中添加一段url全局映射的代码如下。

```kotlin
// 映射的map
val urlMaps = mapOf(
  "https://repo.maven.apache.org/maven2" to "https://maven.aliyun.com/repository/public",
  "https://repo1.maven.apache.org/maven2" to "https://mirrors.cloud.tencent.com/nexus/repository/maven-public/",
  "https://dl.google.com/dl/android/maven2" to "https://mirrors.cloud.tencent.com/nexus/repository/maven-public/",
  "https://plugins.gradle.org/m2" to "https://maven.aliyun.com/repository/gradle-plugin"
)

// 遍历所有仓库并替换
fun RepositoryHandler.enableMirror() {
  all {
    if (this is MavenArtifactRepository) {
      val originalUrl = this.url.toString().removeSuffix("/")
      urlMaps[originalUrl]?.let {
        logger.lifecycle("Repository[$url] is mirrored to $it")
        this.setUrl(it)
      }
    }
  }
}
gradle.allprojects {
  buildscript {
    repositories.enableMirror()
  }
  repositories.enableMirror()
}

gradle.beforeSettings {
  pluginManagement.repositories.enableMirror()
  dependencyResolutionManagement.repositories.enableMirror()
}

```

这个是针对单个项目的，如果要在全部的项目中生效可以将上述代码放入`$GRADLE_USER_HOME/init.gradle.kts`也就是`~/.gradle/init.gradle.kts`文件中,然后开始享受吧。

## 如何全局管理Gradle的依赖

在gradle项目中如果有多个模块，那么在没有统一管理之前，会在每个模块中配置依赖，难免后面升级依赖的时候把某个模块忘了，导致依赖冲突，也比较难以管理依赖。这个时候就要借助Gralde的`dependencyResolutionManagement`了。

1. 先打开`settings.gradle.kts`文件，然后添加如下代码。

```kotlin
dependencyResolutionManagement {}
```

2. 创建`versionCatalogs`

```kotlin
dependencyResolutionManagement {
    versionCatalogs{}
    }
```

3. 创建gradle plugin的依赖

```kotlin
dependencyResolutionManagement {
    versionCatalogs{
        create("plugins") {
            plugin("spring-boot", "org.springframework.boot").version("3.4.0")
            plugin("spring-dependency-management", "io.springdependency-management").version("1.1.6")
            plugin("graalvm.buildtools", "org.graalvm.buildtools.native").version("0.10.3")
        }
    }
}
```

4. 创建普通的依赖

```kotlin
dependencyResolutionManagement {
    versionCatalogs{
        create("spring") {
            version("spring-boot", "3.4.0")
            library("spring-boot-devtools", "org.springframework.boot", "spring-boot-devtools").versionRef("spring-boot")
        }

        create("utils") {
          library("hutool-all", "cn.hutool", "hutool-all").version("5.8.16")
        }
    }
}
```

5. 使用定义的依赖

```kotlin
plugins {
  java
  alias(plugins.plugins.spring.boot)
  alias(plugins.plugins.spring.dependency.management)
  alias(plugins.plugins.graalvm.buildtools)
}
...
dependencies {
  implementation(utils.hutool.all)
  developmentOnly(spring.spring.boot.devtools)
}
```

完整的例子看这里[HyperLinkStretch](https://github.com/rerubbish/HyperLinkStretch),当然这个只是简单的例子，复杂的项目可能需要根据实际情况进行修改。

## 结语

看完之后感觉如何，有没觉得gradle比maven好玩，更灵活。编译速度也是gradle更快，kotlin的一些框架也无法支持maven作为构建工具了，身为一个javaer，gradle也是不能不学的。
