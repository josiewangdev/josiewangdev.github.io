---
layout: post
title: Deep Linking
tags: Project
categories: Tech
cover: upload/blog_masonry_01.jpg
---

目前处理 deep linking，主要有两种方式：

### Custom URL Scheme

在 universal links 出现之前的很长一段时间里，iOS 上主要通过 custom URL scheme 来实现 deep linking，以及 app 间的通信。
<br>
在 info plist 里设置了自定义 URL后，handle URL 的入口是 app delegate 方法
 application:openURL:sourceApplication:annotation:（iOS 9 开始被 deprecate）或 application:openURL:options:
 （iOS 9 引入，但如果没有实现这个方法，在 iOS 9 上还是会向前兼容 call 老方法，所以一般还是实现老方法）。

{% highlight objc %}
- (BOOL)application:(UIApplication *)application openURL:(NSURL *)url
        sourceApplication:(NSString *)sourceApplication annotation:(id)annotation {
    BOOL handled = NO;
    // code to handle the URL
    return handled;
}
{% endhighlight%}

一个比较完整的 NSURL 可以包含以下部分：
<br>
scheme://user:password@host:port/path?query#fragment。但对于 deep linking 来说大部分时候只需要
 scheme://host/path?query。有时候会省去 path 部分，把 host 直接作为 command，如上文提到的点评的 link；也有些 app
  会省去 query 部分，用 path 传参，更接近 RESTful API 的风格。这取决于具体业务逻辑复杂程度以及 handler 的实现方式。
<br>
有一点需要注意的是，规范的 URL 是 percentage encoded 的，所以取出来的参数需要用 stringByReplacingPercentEscapesUsingEncoding:
 或 stringByRemovingPercentEncoding（iOS 7+）方法 decode。反之，拼 URL 的时候应该使用 stringByAddingPercentEscapesUsingEncoding:
  或 stringByAddingPercentEncodingWithAllowedCharacters:（iOS 7+）方法 encode。
  
### Universal Links

Apple 在 iOS 9 上引入了 universal links，相较 custom URL scheme，universal links 有以下好处：

* Custom URL scheme 因为是自定义的协议，所以在没有安装 app 的情况下是无法直接打开的，而 universal links 本身是一个 HTTP/HTTPS 链接，所以有更好的兼容性。
* 不同的 app 是可以定义相同的 custom URL scheme 的，所以会存在抢占或冲突的问题，而 universal links 是从 server 查询由哪个 app 打开的，所以不存在上述问题。
* Universal links 支持从其他 app 的 MKWebView 或 UIWebView 中跳转到目标 app。
* Universal links 本身可以被搜索引擎索引。

<br>
Universal links 的具体实现可以参考官方文档：Support Universal Links。简单来说你需要：

* 添加一个 apple-app-site-association 文件到你的网站来描述 URL 和 app 的关联。
* 添加 com.apple.developer.associated-domains entitlement 来指定要从哪些域名查询 universal links support。
* 在 app delegate 的 application:continueUserActivity:restorationHandler: 方法中 handle userActivity.webpageURL。

处理 URL 本身的方法跟前面处理 custom URL 类似，不再赘述。

[[Ref.] Deferred Deep Linking in iOS](https://tech.glowing.com/cn/deferred-deep-linking-and-branch-sdk-in-ios/)
