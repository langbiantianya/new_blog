---
title: java或者kotlin在gradle中如何使用grpc
published: 2025-05-13
tags: ['java','kotlin','idea','gradle']
description: '都挺好的，安卓也可以用'
category: '技术'
---

## gralde 配置

### kotlin 要使用 grpc 需要配置ksp这个插件所以第一步就是添加ksp插件

```kotlin
plugins {
    id("com.google.devtools.ksp") version "xxx.xxx"
}
```

### 添加protobuf插件

```kotlin
plugins {
    id("com.google.devtools.ksp") version "xxx.xxx"
    id("com.google.protobuf") version "xxx.xxx"
}
```

### 添加grpc的依赖

这里要特别注意下grpc-netty中的netty可能会和其他依赖中的netty冲突，需要你自己排除掉其中一个的netty。

```kotlin
dependencies {
    implementation("io.grpc:grpc-core:last.release")
    runtimeOnly("io.grpc:grpc-netty:last.release")
    implementation("io.grpc:grpc-stub:last.release")
    implementation("io.grpc:grpc-kotlin-stub:last.release")
    implementation("io.grpc:grpc-protobuf:last.release")
    implementation("io.grpc:protoc-gen-grpc-kotlin:last.release")
    implementation("com.google.protobuf:protobuf-java-util:last.release")
    implementation("com.google.protobuf:protobuf-kotlin:last.release")
}
```

### 添加相关配置用来生成grpc的java代码

```kotlin
import com.google.protobuf.gradle.id
......
protobuf {
    protoc {
        artifact = "com.google.protobuf:protoc:4.27.0"
    }
    plugins {
        id("grpc") {
            artifact = "io.grpc:protoc-gen-grpc-java:1.64.0"
        }
        id("grpckt") {
            artifact = "io.grpc:protoc-gen-grpc-kotlin:1.3.0:jdk8@jar"
        }
    }
    generateProtoTasks {
        all().forEach {
            it.plugins {
                id("grpc")
                id("grpckt")
            }
            it.builtins {
                id("kotlin")
            }
        }
    }

    sourceSets {
        main {
            proto {
                srcDir("src/main/resources/protos") // 模块下的proto文件夹
//        include("**/*.proto") 或者这样
            }
        }
    }

}
```

### 完整的配置

```kotlin
import com.google.protobuf.gradle.id
......

plugins {
    id("com.google.devtools.ksp") version "xxx.xxx"
    id("com.google.protobuf") version "xxx.xxx"
    ......
}

......
protobuf {
    protoc {
        artifact = "com.google.protobuf:protoc:4.27.0"
    }
    plugins {
        id("grpc") {
            artifact = "io.grpc:protoc-gen-grpc-java:1.64.0"
        }
        id("grpckt") {
            artifact = "io.grpc:protoc-gen-grpc-kotlin:1.3.0:jdk8@jar"
        }
    }
    generateProtoTasks {
        all().forEach {
            it.plugins {
                id("grpc")
                id("grpckt")
            }
            it.builtins {
                id("kotlin")
            }
        }
    }

    sourceSets {
        main {
            proto {
                srcDir("src/main/resources/protos") // 模块下的proto文件夹
//        include("**/*.proto") 或者这样
            }
        }
    }

}

```
