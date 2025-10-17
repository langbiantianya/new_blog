---
title: Svelte邂逅Web Components了解前端趋势
published: 2025-01-19
tags: ['前端','Svelte','Web Components']
description: '前端趋势回归初心'
category: '杂谈'
---

# Svelte邂逅Web Components了解前端趋势

## 介绍

IE的退出为底层前端技术的发展注入了新活力。以往，为了确保在不同浏览器上的兼容性，Vue与React等主流框架引入了大量中间层来实现功能，这导致各框架构建了复杂的渲染生命周期，学习难度大，技术栈转换也不够便捷。但如今，随着新浏览器API的不断涌现以及ECMAScript标准与W3C标准的持续演进，原本依赖中间层实现的功能正逐步得到浏览器的原生支持。  
在这一趋势下，SvelteJS等框架脱颖而出。它没有在原生浏览器生命周期中添加过多额外的生命周期与上下文，却能实现与Vue和React相媲美的功能。SvelteJS的学习曲线较为平缓，非常适合刚学完HTML、CSS、JavaScript的开发者进阶学习。而且，熟悉SvelteJS后，再切换到Vue或React也会相对容易，因此越来越受到开发者的关注和喜爱。在编译时就将html、js、css一次性编译好，而不需要像vue与react一样在带一个运行时，生为后来者的他避开了以前框架的坑，站在了巨人的肩膀上，目前Start数量已经与3大框架差不了太多了。  
Web Component 标准的诞生为前端开发者带来了巨大便利，然而其问世时机稍显尴尬，恰逢 Vue、React、Angular 等框架风头正劲，导致其光芒被掩盖，至今仍就不温不火。但 Web Component 拥有独特优势，它基于浏览器接口实现，与所使用的 JavaScript 框架无关，只需编写一次组件，便可在多种框架中无缝使用。

## 上手

如果你有学过vuejs2那么你上手的时候会非常熟悉svelte的语法以及声明式的绑定，例如下面这段代码[+layout.svelte](https://github.com/langbiantianya/cherry-markdown-webview/blob/main/frontend/src/routes/%2Blayout.svelte) 中的`personalizaDrawer`。其代码分为3块script、html、css这3块，设计与vue有着很大相似之处，代码清晰明了。当如如果想具体了解学习还是要去[svelte.dev](https://svelte.dev/)，里面有交互式学习很容上手的，概念也不多都是很目前很成熟的一些东西。

```svelte
<script>
 import { goto } from '$app/navigation';
 import '../app.css';
 import { Circle2 } from 'svelte-loading-spinners';
 import { globalState } from '$lib/store';
 import { EventsOn, EventsOff } from '$lib/wailsjs/runtime';
 import '@fluentui/web-components/drawer.js';
 import '@fluentui/web-components/button.js';
 import '@fluentui/web-components/dialog.js';
 import Options from '$lib/components/options/index.svelte';
 import Personaliza from '$lib/components/personaliza/index.svelte';
 import About from '$lib/components/about/index.svelte';
 import { GetActivatedTheme } from '$lib/wailsjs/go/main/App';

 import { onMount } from 'svelte';
 import { changeMainThemeEvent, loadBackgroundImage } from '$lib/theme';

 /** @type {{children: import('svelte').Snippet}} */
 let { children } = $props();
 /** @type {import('@fluentui/web-components').Drawer} */
 let optionsDrawer;

 /** @type {import('@fluentui/web-components').Drawer} */
 let personalizaDrawer;

 /** @type {import('@fluentui/web-components').Dialog} */
 let aboutDialog;
 function heidOptionsDrawer() {
  optionsDrawer.hide();
 }

 function heidPersonalizaDrawer() {
  personalizaDrawer.hide();
 }
 try {
  EventsOff('optionsEvent', 'personalizaEvent', 'aboutEvent');
  EventsOn('aboutEvent', (event) => {
   aboutDialog.show();
  });
  EventsOn('optionsEvent', (event) => {
   optionsDrawer.show();
   personalizaDrawer.hide();
   aboutDialog.hide();
  });
  EventsOn('personalizaEvent', (event) => {
   personalizaDrawer.show();
   optionsDrawer.hide();
   aboutDialog.hide();
  });
 } catch (error) {
  console.error(error);
 }

 onMount(async () => {
  changeMainThemeEvent(await GetActivatedTheme());
 });
</script>

<div class="app h-full">
 <div class="background-image"></div>
 <main class="h-full">
  {@render children()}
 </main>
 <footer></footer>
 <fluent-drawer class="m-0 p-0" type="model" position="end" size="full" bind:this={optionsDrawer}>
  <Options heid={heidOptionsDrawer} />
 </fluent-drawer>

 <fluent-drawer
  class="m-0 p-0"
  type="model"
  position="end"
  size="full"
  bind:this={personalizaDrawer}
 >
  <Personaliza heid={heidPersonalizaDrawer} />
 </fluent-drawer>
 <fluent-dialog bind:this={aboutDialog}>
  <About />
 </fluent-dialog>

 {#if $globalState.loading}
  <div class="background-image"></div>
  <div
   class="loading bg-base absolute left-0 right-0 top-0 z-50 flex h-lvh w-full items-center justify-center"
  >
   <Circle2 size="50" unit="lvh" />
  </div>
 {/if}
</div>

<style>
 main {
  opacity: var(--edit-opacity);
 }
 .loading {
  opacity: var(--edit-opacity);
 }
 .background-image {
  opacity: 1;
  position: absolute;
  top: 0;
  left: 0;
  width: 100lvw;
  height: 100lvh;
  background-image: var(--background-image); /* 设置背景图片路径 */
  background-size: cover; /* 调整背景图片大小以覆盖整个容器 */
  background-position: center; /* 居中背景图片 */
 }
</style>
```

在上面这段代码中使用了Microsoft开源的fluentui的web component组件，以`dialog`举例，导入依赖后，使用起来就和普通的组件一样`<fluent-dialog></fluent-dialog>`。上面这段代码还是比较复杂的，来看个干净简单一点的吧[index.html](https://github.com/rerubbish/HyperLinkStretch/blob/master/src/main/resources/static/index.html)。里头只用了原生的html与js代码。其中的`fluent-button`与`fluent-text-area`全是使用fluentui的web-components组件，当然感兴趣的话可以去看看fluentui的组件源码。第一次使用的时候我也是一头雾水无从下手，但是在查看了fluentui 组件定义的index.d之后逐渐会用了。

```html
<!DOCTYPE html>
<html lang="zh-cn">

<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, user-scalable=no">
    <script src="https://cdnjs.cloudflare.com/ajax/libs/qrcodejs/1.0.0/qrcode.min.js">

    </script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/clipboard.js/2.0.11/clipboard.min.js"></script>
    <script type="module" src="https://unpkg.com/@fluentui/web-components">
    </script>
    <title>HyperLinkStretch</title>
    <style>
        .content {
            display: flex;
            flex-direction: column;
            justify-content: center;
            align-content: center;
            align-items: center;
            width: 100%;
            height: 95vh;
        }

        .input {
            width: 100%;
            display: flex;
            align-content: center;
            align-items: center;
            justify-content: center;
            flex-wrap: wrap;
        }

        .input_area {
            max-height: 2.5rem;
            max-width: 40rem;
            min-width: 13rem;
            flex: 5;
            margin-bottom: 0.5rem;
        }

        .input_button {
            height: 2.5rem;
            flex: 1;
            max-width: 5rem;
            min-width: 3rem;
            margin-bottom: 0.5rem;
        }

        .result {
            width: 100%;
        }

        .result_card {
            margin: 0 auto;
            max-width: 46rem;
            min-width: 13rem;
            padding: 0.5rem;
        }

        .result_qr img {
            width: 100%;
            height: auto;
        }

        .result_qr {
            width: min(90svw, 90svh);
            height: min(90svw, 90svh);
            padding: 1rem;
            position: absolute;
            left: 50%;
            top: 50%;
            transform: translate(-50%, -50%);
        }

        .result_link {
            flex: 5;
            max-width: 100%;
            max-height: 10rem;
            padding-left: 1rem;
            overflow-y: scroll;
        }

        .result_link_url {
            word-wrap: break-word;
        }

        .result_button {
            flex: 3;
            position: relative;
            right: 0.5rem;
            text-align: right;
            height: 2rem;
            min-width: 10rem;
            /* white-space: nowrap; */
        }

        .result_tool {
            display: flex;
            flex-wrap: wrap;
            justify-content: space-between;
            align-items: center;
            padding: 1rem;
            padding-bottom: 0.5rem;
        }

        a {
            display: inline-block;
            padding: 10px 20px;
            color: #000;
            text-decoration: none;
            transition: 0.3s ease;
        }

        a:hover {
            color: #0473ce;
            text-shadow: -1px -1px 0 #fff, 1px -1px 0 #fff, -1px 1px 0 #000, 1px 1px 0 #000;
        }

        @media (max-width: 41rem) {
            .content {
                height: auto;
            }

            .result_link {
                max-height: 100%;
                overflow-y: auto;
            }

            .result_tool {
                justify-content: center;
            }

            #show_qr {
                display: none;
            }

            #copy_result_link_url {
                width: 100%;
                margin-left: 0.5rem;
            }

            .skeleton {
                display: none;
            }

            .result_button {
                text-align: center;
                max-height: 100%;
                flex: 1 0 100%;
            }

            .input_area {
                max-height: 100%;
                min-height: 2.5rem;
            }

            .result_link {
                max-width: 40rem;
                min-width: 13rem;
                flex: 1 0 100%;
            }

            .input_button {
                max-width: 40rem;
                min-width: 13rem;
                flex: 1 0 100%;
            }
        }
    </style>
</head>

<body>
<script type="text/javascript">
    function isValidUrl(url) {
        try {
            new URL(url);
            return true;
        } catch (_) {
            return false;
        }
    }

    async function generate(url) {
        isValidUrl(url) || window.alert("请输入正确的url")
        const response = await fetch(`/api/v1/generate?url=${url}`, {
            method: 'GET',
            headers: {
                'Content-Type': 'application/json',
            }
        })
        const result = await response.json()
        var currentUrl = window.location.href;
        // 使用 URL 对象来解析 URL
        var url = new URL(currentUrl);

        // 从 URL 对象中提取主机名
        var host = url.hostname;
        // 打印主机名
        const longUrl = `${window.location.protocol}//${host}${window.location.port ? ":" + window.location.port : ""}${result.targetURL}`
        document.getElementById("result_link_url").innerText = longUrl
        document.getElementById("qrcode").innerHTML=""
        new QRCode(document.getElementById("qrcode"), {
            text: longUrl,
            width: 2048,
            height: 2048,
            colorDark: "#000000",
            colorLight: "#ffffff",
            correctLevel: QRCode.CorrectLevel.H
        });
        document.getElementById("result").hidden = false;
        document.getElementById("copy_result_link_url").setAttribute("data-clipboard-text", longUrl)
    }

    window.onload = () => {
        document.getElementById("content").addEventListener("click", (event) => {
            const url = document.getElementById("qrcode").hidden = true
        })
        document.getElementById("show_qr").addEventListener("click", (event) => {
            event.stopPropagation()
            const url = document.getElementById("qrcode").hidden = false
        })
        document.getElementById("qrcode").addEventListener("click", (event) => {
            const url = document.getElementById("qrcode").hidden = true
        })
        document.getElementById("input_button").addEventListener("click", (event) => {
            const url = document.getElementById("input_area").currentValue
            generate(url)
        })
        var clipboard = new ClipboardJS('#copy_result_link_url');

        clipboard.on('success', function (e) {
            console.info('Action:', e.action);
            console.info('Text:', e.text);
            console.info('Trigger:', e.trigger);
            window.alert("copy success")
        });

        clipboard.on('error', function (e) {
            console.error('Action:', e.action);
            console.error('Trigger:', e.trigger);
            window.alert("copy error")
        });
    }
</script>
<div id="content" class="content">
    <fluent-tooltip id="tooltip" anchor="anchor-default">
        点击前往github仓库
    </fluent-tooltip>
    <h1 id="anchor-default" aria-describedby="tooltip"><a
            href="https://github.com/langbiantianya/HyperLinkStretch">一个长链</a></h1>
    <div class="input">
        <fluent-text-area id="input_area" class="input_area"
                          placeholder="请输入包含”http://“或“https://”网址"></fluent-text-area>
        <fluent-skeleton class="skeleton" style="width: 1rem;" shape="rect" shimmer="false"></fluent-skeleton>
        <fluent-button id="input_button" class="input_button" appearance="accent">生成</fluent-button>
    </div>

    <div id="result" class="result" hidden>
        <fluent-card class="result_card">
            <div class="result_tool">
                <span style="line-height: 2rem; flex: 8;">该应用用于娱乐用途，请勿在生产上使用。</span>
                <div class="result_button">
                    <fluent-button id="show_qr">二维码</fluent-button>
                    <fluent-button id="copy_result_link_url" appearance="accent" data-clipboard-text=""
                                   data-clipboard-action="copy">复制链接
                    </fluent-button>
                </div>
            </div>

            <div class="result_link">
                <span id="result_link_url" class="result_link_url"></span>
            </div>
    </div>
    </fluent-card>
</div>
<div id="qrcode" class="result_qr"></div>
</body>

</html>
```

## 实际开发的例子

例子的话上面2块代码的出处就是我写的2个项目。  
其中比较纯粹的web component请看这个[HyperLinkStretch](https://github.com/rerubbish/HyperLinkStretch)一个简单的Demo。  
结合了svelte与web component的请看这个[cherry-markdown-webview](https://github.com/langbiantianya/cherry-markdown-webview/tree/main/frontend)复杂一些的项目，目前还在持续开发中。  

## 个人认为的未来趋势

国内的大环境目前还是很烂的，大多以外包为主，各种admin框架已经定死前端框架，国内领导层年纪都挺大的也不愿意了解新技术跟换技术栈，稳定大于一切，加上国内也没有大厂站台与推广，想真正流行起来难度还是很大的。我个人认为未来的趋势应该还是会退却多余的中间层而使用新的api，当然像React这种按需渲染的性能还是很诱人的。像是Svelte这种框架倒是很适合不怎么写前端的后端使用例如我，可以了解前端技术的变化以写出更符合前端使用习惯的接口。正所谓不会写前端的后端不是好后端，不会写后端的前端不是好前端。
