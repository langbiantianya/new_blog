---
title: 我如何实现外包的java项目生成器
published: 2025-07-20
tags: ['golang','wsam','java','gradle']
description: '通过是golang编写生成器的逻辑然后编译为wasm在web上运行'
category: '技术'
---

## 为何做出这样的技术选型

由于我自己的服务器资源紧张，也不想在这上面多花钱，就想到webassembly这个技术。全部编译成静态资源托管到cloudflare的page上面，就如同我这个博客一样。 面对这样的需求我一下子就想到对wasm支持比较好的rust、golang和kotlin，但是由于我对rust不怎么感兴趣也没学过就成了golang与kotlin二选一的结局了。最后考虑到kotlin跨平台目前使用的前端UI还是jetpack，我自己对android开发不熟悉，最后选了golang的go-app来实现这一需求。至于js呢，js对于二进制文件的处理以及io方面的操作不是很方便，加上go标准库的优秀以及wasm运行效率上区别，还有就是我想用些有趣的方式去实现。

## 重要功能的实现

### 模板

模板资源就是一个个的特殊的java项目，里面的关键位置使用了golang标准库中的template来实现替换，并在编译时将这些文件一起编译到可执行文件中。在调用替换方法后会将前端传来的参数在里面一一替换。  
不同的模板自己实现自己的生成逻辑，最后统一实现生成函数接口，给调用者。

#### 示例代码

##### 统一生成zip压缩包二进制文件接口

```golang

type TemplateData interface {
 GenZip() ([]byte, error)
}
```

```golang
func (data WebTemplateData) GenZip() ([]byte, error) {
 var buf bytes.Buffer
 zw := zip.NewWriter(&buf)
 defer zw.Close()

 // 遍历assets/gradle/web/2目录下所有文件
 err := fs.WalkDir(distFS, fmt.Sprintf("assets/gradle/web/%d", data.SpringBootVersion), func(path string, d fs.DirEntry, walkErr error) error {
  if walkErr != nil {
   return walkErr
  }

  // 计算相对路径（去除assets/gradle/web/2前缀）
  relPath := strings.TrimPrefix(path, fmt.Sprintf("assets/gradle/web/%d/", data.SpringBootVersion))
  if relPath == path { // 处理根目录情况
   relPath = ""
  }

  // 替换文件路径
  if strings.Contains(relPath, "moduleName") {
   relPath = strings.ReplaceAll(relPath, "moduleName", data.ModuleName)
  }

  if d.IsDir() {
   // 跳过根目录的创建（relPath为空时不创建）
   if relPath != "" {
    // 创建目录条目（zip目录需要以/结尾）
    _, err := zw.Create(relPath + "/")
    return err
   }
   return nil
  }
  // 判断文件是否需要替换
  var fileData []byte
  if strings.Contains(relPath, "build.gradle.kts") {
   strData, err := data.GenBuildKts()
   if err != nil {
    return err
   }
   fileData = []byte(strData)
  } else if strings.Contains(relPath, "settings.gradle.kts") {
   strData, err := data.GenSettingsKts()
   if err != nil {
    return err
   }
   fileData = []byte(strData)
  } else if strings.Contains(relPath, "application.properties") {
   strData, err := data.GenApplication()
   if err != nil {
    return err
   }
   fileData = []byte(strData)
  } else if strings.Contains(relPath, "run.sh") {
   strData, err := data.GenRunSh()
   if err != nil {
    return err
   }
   fileData = []byte(strData)
  } else if strings.Contains(relPath, "start_web.sh") {
   strData, err := data.GenStartWebSh()
   if err != nil {
    return err
   }
   fileData = []byte(strData)
  } else if strings.Contains(relPath, "dk_product_component.yaml.j2") {
   strData, err := data.GenBlueprint()
   if err != nil {
    return err
   }
   fileData = []byte(strData)
  } else if strings.Contains(relPath, "README.md") {
   strData, err := data.GenReadme()
   if err != nil {
    return err
   }
   fileData = []byte(strData)
  } else {
   // 读取文件内容
   fileDataS, err := distFS.ReadFile(path)
   if err != nil {
    return err
   }
   fileData = fileDataS
  }

  info, err := d.Info()
  if err != nil {
   return err
  }
  // 创建文件头
  fh, err := zip.FileInfoHeader(info)
  if err != nil {
   return err
  }
  fh.Name = relPath       // 设置文件在zip中的路径
  fh.Method = zip.Deflate // 使用默认压缩算法

  // 写入文件内容
  fileWriter, err := zw.CreateHeader(fh)
  if err != nil {
   return err
  }
  _, err = fileWriter.Write(fileData)
  return err
 })

 if err != nil {
  return nil, err
 }

 // 确保所有数据写入buffer
 if err := zw.Close(); err != nil {
  return nil, err
 }

 return buf.Bytes(), nil
}
```

##### 模板内容替换逻辑

```golang
func (data WebTemplateData) GenBuildKts() (string, error) {
 // 读取嵌入的模板文件
 buildBytes, err := distFS.ReadFile(fmt.Sprintf("assets/gradle/web/%d/build.gradle.kts", data.SpringBootVersion))
 if err != nil {
  return "", err // 改为返回错误而非 panic
 }

 // 解析模板内容
 tpl, err := template.New("buildGradle").Parse(string(buildBytes))
 if err != nil {
  return "", err
 }

 var result strings.Builder
 if err := tpl.Execute(&result, data); err != nil {
  return "", err
 }
 return result.String(), nil // 返回生成后的内容和错误

}
```

### UI

UI使用了go-app这个项目来操作dom元素，其中的难点是css样式和二进制文件保存。其中样式借助tailwind来写组件，再将他们排列组合。二进制文件的保存就很麻烦了，首先在dom树中添加了一个a标签，url上填入了压缩包的base64编码的字符串，在模拟点击触发使得浏览器下载压缩包，然后再移除这个元素。

#### 示例代码

##### UI 组件

以button为例看起来其实和大部分的前端ui框架差不多

```golang

package compose

import "github.com/maxence-charriere/go-app/v10/pkg/app"

// Button 通用按钮组件（Tailwind版，类名拆分）
// btnType 可选值："primary"（默认）、"secondary"、"tertiary"、"warning"、"danger"
func Button(name, btnType string) app.HTMLButton {
 button := app.Button().
  Type("submit").
  Text(name).
  // 通用基础样式（每个类名单独作为字符串参数）
  Class(
   "rounded-md",
   "px-4",
   "py-2",
   "text-sm",
   "font-medium",
   "border",
   "border-transparent",
   "cursor-pointer",
   "transition-all",
   "duration-200",
   "ease-in-out",
  )

 // 根据类型设置具体样式（每个类名单独作为字符串参数）
 switch btnType {
 case "primary": // 主按钮
  button = button.Class(
   "bg-[#5072a7]",
   "text-white",
   "hover:bg-[#466492]",
   "active:bg-[#3d577c]",
  )

 case "secondary": // 次要按钮
  button = button.Class(
   "bg-white",
   "text-[#6c757d]",
   "border-[#6c757d]",
   "hover:bg-[#f8f9fa]",
   "active:bg-[#e9ecef]",
  )

 case "tertiary": // 第三级按钮
  button = button.Class(
   "bg-transparent",
   "text-[#5072a7]",
   "hover:bg-[#f0f4f8]",
   "active:bg-[#e6edf3]",
  )

 case "warning": // 警告按钮
  button = button.Class(
   "bg-[#ffc107]",
   "text-[rgba(0,0,0,0.85)]",
   "hover:bg-[#ffca2c]",
   "active:bg-[#ffb700]",
  )

 case "danger": // 危险按钮
  button = button.Class(
   "bg-[#dc3545]",
   "text-white",
   "hover:bg-[#e04b59]",
   "active:bg-[#c82333]",
  )

 default: // 默认主按钮
  button = button.Class(
   "bg-[#5072a7]",
   "text-white",
   "hover:bg-[#466492]",
   "active:bg-[#3d577c]",
  )
 }
 return button
}

```

##### 操作dom元素保存二进制文件

```golang
package template

import (
 "encoding/base64"

 "github.com/maxence-charriere/go-app/v10/pkg/app"
)

func SaveZipDataLocally(ctx app.Context, e app.Event, zipBytes []byte, fileName string) {

 // 将ZIP文件内容编码为Base64
 b64Data := base64.StdEncoding.EncodeToString(zipBytes)

 // 创建Data URL
 dataURL := "data:application/zip;base64," + b64Data

 // 确保DOM操作在安全的上下文中执行
 ctx.Defer(func(ctx app.Context) {
  // 创建下载链接
  a := app.Window().Get("document").Call("createElement", "a")
  a.Set("href", dataURL)
  a.Set("download", fileName)

  // 添加到DOM并触发点击
  app.Window().Get("document").Get("body").Call("appendChild", a)
  a.Call("click")

  // 移除链接
  a.Call("remove")

 })
}

```

## 积累

1. 熟悉goalng的一些偏门用法。
2. 更加熟悉外包项目中的模板渲染。
3. 脱离框架后发现自己还挺行的。

## 踩坑

1. 前端组建和样式太难写了，最后还是借助了tailwind来写样式。
2. wasm中调用浏览器事件的难度超乎我的想象。
3. 野心太大想要同时维护native app和web app，导致工作量太大了，使得功能迭代太慢了。
4. 互联网上可以借鉴的文章太少了只能自己一点点尝试。

## 有成功帮我摸到鱼吗

目前来说有但是不多，摸鱼的时间被我拿去优化这个生成器和学习zig去了。虽然提高了工作效率，但是也有可能导致需要完成的工作量变得更多，工资也不会涨。

## 吐槽

用vscode在windwos上写go是真的卡，我自己电脑的配置本身也不低了，对比goland和zed简直卡暴了，开箱即用还得是goland。

## 项目地址

[gendk](https://github.com/langbiantianya/gendk)
