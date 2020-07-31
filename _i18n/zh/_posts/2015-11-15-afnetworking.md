---
layout: post
title: AFNetworking
tags: iOS-Framework
categories: Dev
description: AFNetworking使用
---

## AFNetworking

---

使用语言: **Objective-C**

---

> #### 实际应用 {#usage}

#### CORE:

AFURLConnectionOperation:一个 NSOperation 实现了NSURLConnection 的代理方法.

<br>

#### HTTP REQUEST:

* AFHTTPRequestOperation

AFURLConnectionOperation的子类,当request使用的协议为HTTP和HTTPS时,它压缩了用于决定request是否成功的状态码和内容类型.

* AFJSONRequestOperation

AFHTTPRequestOperation的一个子类,用于下载和处理jason response数据.

* AFXMLRequestOperation

AFHTTPRequestOperation的一个子类,用于下载和处理xml response数据.

* AFPropertyListRequestOperation

AFHTTPRequestOperation的一个子类,用于下载和处理property list response数据.

<br>

#### HTTP CLIENT:

**AFHTTPClient 捕获一个基于http协议的网络应用程序的公共交流模式包含:**

* 使用基本的url相关路径来只做request
* 为request自动添加设置http headers.
* 使用http 基础证书或者OAuth来验证request
* 为由client制作的requests管理一个NSOperationQueue
* 从NSDictionary生成一个查询字符串或http bodies.
* 从request中构建多部件
* 自动的解析http response数据为相应的表现数据
* 在网络可达性测试用监控和响应变化.

<br>

#### IMAGES

* AFImageRequestOperation

一个AFHTTPRequestOperation的子类,用于下载和处理图片.

* UIImageView+AFNetworking

添加一些方法到UIImageView中,为了从一个URL中异步加载远程图片

<br>

#### XML REQUEST

```objc
NSURLRequest *request = [NSURLRequest requestWithURL:[NSURL URLWithString:@"http://api.flickr.com/services/rest/?method=flickr.groups.browse&api_key=b6300e17ad3c506e706cb0072175d047&cat_id=34427469792%40N01&format=rest"]];
AFXMLRequestOperation *operation = [AFXMLRequestOperation XMLParserRequestOperationWithRequest:request success:^(NSURLRequest *request, NSHTTPURLResponse *response, NSXMLParser *XMLParser) {
    XMLParser.delegate = self;	
    [XMLParser parse];
} failure:nil];	
[operation start];
```

<br>

#### IMAGE REQUEST

```objc
UIImageView *imageView = [[UIImageView alloc] initWithFrame:CGRectMake(0.0f, 0.0f, 100.0f, 100.0f)];
[imageView setImageWithURL:[NSURL URLWithString:@"http://i.imgur.com/r4uwx.jpg"] placeholderImage:[UIImage imageNamed:@"placeholder-avatar"]];
```

<br>

#### API CLIENT REQUEST

```objc
[[AFGowallaAPIClient sharedClient] getPath:@"/spots/9223" parameters:nil success:^(AFHTTPRequestOperation *operation, id responseObject) {
    NSLog(@"Name: %@", [responseObject valueForKeyPath:@"name"]);
    NSLog(@"Address: %@", [responseObject valueForKeyPath:@"address.street_address"]);
} failure:nil];
```

<br>

#### FILE UPLOAD WITH PROGRESS CALLBACK

```objc
NSURL *url = [NSURL URLWithString:@"http://api-base-url.com"];
AFHTTPClient *httpClient = [[AFHTTPClient alloc] initWithBaseURL:url];
NSData *imageData = UIImageJPEGRepresentation([UIImage imageNamed:@"avatar.jpg"], 0.5);
NSMutableURLRequest *request = [httpClient multipartFormRequestWithMethod:@"POST" path:@"/upload" parameters:nil constructingBodyWithBlock: ^(id <AFMultipartFormData>formData) {
    [formData appendPartWithFileData:imageData name:@"avatar" fileName:@"avatar.jpg" mimeType:@"image/jpeg"];
}];
 
AFHTTPRequestOperation *operation = [[[AFHTTPRequestOperation alloc] initWithRequest:request] autorelease];
[operation setUploadProgressBlock:^(NSInteger bytesWritten, long long totalBytesWritten, long long totalBytesExpectedToWrite) {
    NSLog(@"Sent %lld of %lld bytes", totalBytesWritten, totalBytesExpectedToWrite);
}];
[operation start];
```

<br>

#### STREAMING REQUEST

```objc
NSURLRequest *request = [NSURLRequest requestWithURL:[NSURL URLWithString:@"http://localhost:8080/encode"]];
AFHTTPRequestOperation *operation = [[[AFHTTPRequestOperation alloc] initWithRequest:request] autorelease];
operation.inputStream = [NSInputStream inputStreamWithFileAtPath:[[NSBundle mainBundle] pathForResource:@"large-image" ofType:@"tiff"]];
operation.outputStream = [NSOutputStream outputStreamToMemory];
[operation start];
```
