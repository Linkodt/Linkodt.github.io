---
title: Hybird技术学习
date: 2025-10-28 09:32:47
tags: 
    - 前端
    - 移动端
---
# Hybird技术学习

## URL Scheme拦截

这是最基础、兼容性最好的通信方式。

### 底层原理

**核心思想：** 利用 WebView 的 URL 加载机制，通过创建一个特殊格式的 URL 来"传递消息"，原生代码通过拦截并解析这个 URL 来获取指令。

URL Scheme拦截的核心原理：**在WebView中发出的网络请求，客户端都能进行监听和捕获**。通过定制特定的URL Scheme规则，使H5页面能够通过特定格式的URL触发Native端的逻辑处理。

### 通信流程

1.  **H5发起请求**：使用创建隐藏iframe的方式发送请求（避免location.href并发请求被合并的问题）

    ```javascript
    function iosExecute(action, param) {
        param['methodName'] = action;
        let iframe = env.createIframe();
        let paramStr = JSON.stringify(param);
        iframe.src = `zznative://zhuanzhuan.hybrid.ios/?infos=${encodeURIComponent(paramStr)}`;
        document.body.appendChild(iframe);
        setTimeout(() => iframe.remove(), 300);
    }
    ```
2.  **Native拦截请求**：通过WebView的回调方法捕获并解析URL
3.  **执行业务逻辑**：根据URL中的协议头和参数执行对应功能
4.  **回调结果**：Native处理完成后，可通过JSBridge机制将结果返回给H5

### 优势and劣势

优势：简单、适用

劣势：安全、参数长度、用户体验差

## JS birdge注入

### 核心原理

**JavaScript注入的本质是：Native端在JavaScript运行环境中动态注入API，使H5页面能够直接调用Native功能**。

Hybrid应用的本质是在原生App中使用WebView作为容器承载Web页面，最核心的点就是Native和H5之间的双向通讯层，即JSBridge。API注入是实现这一通讯的关键方式。

### 两个平台的实现

#### 安卓

在Android中，通过`addJavascriptInterface`方法将Java对象注入到JavaScript环境中：

```java
webView.addJavascriptInterface(new JavaScriptInterface(), "Android");

// JavaScriptInterface类实现
public class JavaScriptInterface {
    @JavascriptInterface
    public void callNativeMethod(String param) {
        // Native逻辑处理
    }
}
```

在H5页面中，可以通过`window.Android`调用注入的方法：

```javascript
javascript编辑// H5端调用
window.Android.callNativeMethod("Hello from H5");
```

#### IOS

在iOS中，通过WKWebView或UIWebView实现注入：

```c
let userContentController = WKUserContentController()
userContentController.add(self, name: "iOS")
let config = WKWebViewConfiguration()
config.userContentController = userContentController
webView = WKWebView(frame: .zero, configuration: config)

// 实现WKScriptMessageHandler协议
func userContentController(_ userContentController: WKUserContentController, didReceive message: WKScriptMessage) {
    if message.name == "iOS" {
        // 处理来自JS的消息
    }
}
```

在H5页面中，通过`webkit.messageHandlers`调用：

```javascript
javascript编辑// H5端调用
webkit.messageHandlers.iOS.postMessage({data: "Hello from H5"});
```

### 注入的实现流程

1.  **Native端初始化**：在WebView加载页面前，注册需要注入的API对象
2.  **注入过程**：将Native对象注入到JavaScript运行环境
3.  **H5调用**：H5页面通过注入的API调用Native功能
4.  **回调机制**：Native处理完成后，通过回调函数将结果返回给H5

### 优势与局限

#### 优势

1.  **双向通信**：支持Native和H5之间的双向通信
2.  **数据传递能力强**：可以传递复杂对象，不受URL长度限制
3.  **用户体验好**：无需用户确认弹窗
4.  **实现相对简单**：Android和iOS都有官方支持的API

#### 局限

1.  **兼容性问题**：Android 4.2以下版本存在安全漏洞
2.  **安全风险**：若未正确设置`@JavascriptInterface`，可能导致安全漏洞
3.  **API暴露**：所有注入的API都对H5可见，需要谨慎设计

## Native执行JavaScript

### 核心原理

Native执行JavaScript的本质是**通过WebView提供的API，在Native端直接执行JavaScript代码**，使Native能够控制H5页面的行为，实现Native主动调用H5的功能。

### 实现方式

#### 1. iOS平台实现（基于JavaScriptCore）

iOS平台主要通过JavaScriptCore框架实现Native执行JavaScript：

```c
objectivec编辑// 创建JS上下文
JSContext *jsContext = [[JSContext alloc] init];

// 执行无参数的JS代码
NSString *script = @"console.log('Native call JS');";
[jsContext evaluateScript:script];

// 执行有参数的JS代码
NSString *argument = @"Hello from Native";
NSString *jsString = @"function receive(arg) { console.log('Received: ' + arg); }; receive('%@')";
NSString *scriptWithArg = [NSString stringWithFormat:jsString, argument];
[jsContext evaluateScript:scriptWithArg];
```

**实现原理**：

*   通过`JSContext`对象创建JavaScript执行环境
*   使用`evaluateScript`方法执行JS代码
*   可以通过字符串拼接方式传递参数

#### 2. Android平台实现

Android平台主要通过WebView的`evaluateJavascript`方法执行JS代码：

```java
String jsCode = "console.log('Native call JS');";
webView.evaluateJavascript(jsCode, new ValueCallback<String>() {
    @Override
    public void onReceiveValue(String value) {
        // 处理JS返回值
    }
});

// 带参数执行
String arg = "Hello from Native";
String jsWithParam = String.format("function receive(arg) { console.log('Received: ' + arg); }; receive('%s');", arg);
webView.evaluateJavascript(jsWithParam, null);
```

**实现原理**：

*   通过WebView的`evaluateJavascript`方法执行JS代码
*   可以通过回调函数获取JS执行结果
*   需要处理JS返回值

### 与JSBridge的关系

Native执行JavaScript是JSBridge双向通信机制的重要组成部分：

1.  **JSBridge的双向通信**：

    *   H5 → Native：通过URL Scheme、API注入或prompt拦截
    *   Native → H5：通过Native执行JavaScript
2.  **Native执行JS的典型场景**：

    *   Native处理完数据后，需要更新H5页面
    *   Native需要触发H5页面的某些逻辑
    *   Native需要执行H5页面的特定函数

### 详细执行流程

1.  **Native端准备**：

    *   获取或创建JS执行环境（iOS：JSContext；Android：WebView）
    *   准备要执行的JS代码
2.  **参数处理**：

    *   将需要传递的参数格式化为字符串
    *   通过字符串拼接或JSON序列化方式处理复杂参数
3.  **执行JS代码**：

    *   调用`evaluateScript`（iOS）或`evaluateJavascript`（Android）方法
    *   传递JS代码字符串
4.  **处理结果**：

    *   对于Android，通过回调函数获取JS执行结果
    *   对于iOS，可以通过JSContext的`evaluateScript`返回值获取结果

## 消息队列式的 WebView ↔ Native 通信

核心原理：用一个**有序的、带回调 ID 的消息队列**来把 JS 侧的调用打包发送给 Native（或反向），Native 执行后通过回调 ID 将结果送回 JS。队列能保证顺序、支持异步回调、能批量/延迟发送以提高性能，并避免同步阻塞主线程引发死锁。

### 实现要点

#### js端实现要点

要点：维护 `outbox`（待发送队列）和 `callbacks`（回调 map），触发发送的策略可以是立即、微任务合并、定时批量发送、或特定事件（如页面可见）。

#### native端实现要点

*   Native 要能接收批量消息、解析 JSON、按顺序执行对应方法、并将响应打包发回 JS。
*   执行会涉及主线程（UI）与工作线程（I/O/业务），注意线程切换与同步问题。

