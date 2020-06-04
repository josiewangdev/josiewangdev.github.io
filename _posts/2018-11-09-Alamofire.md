---
layout: post
title: "Alamofire"
tags: [iOS]
categories: [Tech]
cover: "upload/blog_masonry_05.jpg"
---

* Alamofire

{% highlight swift %}
Alamofire.request(.GET, "https://httpbin.org/get", parameters: ["foo": "bar"])
    .responseJSON { response in

        print(response.request)  // 请求对象
        print(response.response) // 响应对象
        print(response.data)     // 服务端返回的数据

        if let JSON = response.result.value {
            print("JSON: \(JSON)")
        }

}
// Alamofire 目前支持的数据序列化方法：
// response()
// responseData()
// responseString(encoding: NSStringEncoding)
// responseJSON(options: NSJSONReadingOptions)
// responsePropertyList(options: NSPropertyListReadOptions)
{% endhighlight %}

* 上传文件

{% highlight swift %}
let fileURL = NSBundle.mainBundle().URLForResource("Default", withExtension: "png")
Alamofire.upload(.POST, "https://httpbin.org/post", file: fileURL)

//我们还可以获取上传时候的进度
Alamofire.upload(.POST, "https://httpbin.org/post", file: fileURL)
         .progress { bytesWritten, totalBytesWritten, totalBytesExpectedToWrite in
             print(totalBytesWritten)

             // This closure is NOT called on the main queue for performance
             // reasons. To update your ui, dispatch to the main queue.
             dispatch_async(dispatch_get_main_queue()) {
                 print("Total bytes written on main queue: \(totalBytesWritten)")
             }
         }
         .responseJSON { response in
             debugPrint(response)
         }

//MultipartFormData 形式的表单数据上传
Alamofire.upload(
    .POST,
    "https://httpbin.org/post",
    multipartFormData: { multipartFormData in
        multipartFormData.appendBodyPart(fileURL: unicornImageURL, name: "unicorn")
        multipartFormData.appendBodyPart(fileURL: rainbowImageURL, name: "rainbow")
    },
    encodingCompletion: { encodingResult in
        switch encodingResult {
        case .Success(let upload, _, _):
            upload.responseJSON { response in
                debugPrint(response)
            }
        case .Failure(let encodingError):
            print(encodingError)
        }
    }
)
{% endhighlight %}

* 下载文件

{% highlight swift %}
Alamofire.download(.GET, "https://httpbin.org/stream/100") { temporaryURL, response in
    let fileManager = NSFileManager.defaultManager()
    let directoryURL = fileManager.URLsForDirectory(.DocumentDirectory, inDomains: .UserDomainMask)[0]
    let pathComponent = response.suggestedFilename

    return directoryURL.URLByAppendingPathComponent(pathComponent!)
}

//设置默认的下载存储位置
let destination = Alamofire.Request.suggestedDownloadDestination(directory: .DocumentDirectory, domain: .UserDomainMask)
Alamofire.download(.GET, "https://httpbin.org/stream/100", destination: destination)

//也提供了 progress 方法检测下载的进度
Alamofire.download(.GET, "https://httpbin.org/stream/100", destination: destination)
         .progress { bytesRead, totalBytesRead, totalBytesExpectedToRead in
             print(totalBytesRead)

             dispatch_async(dispatch_get_main_queue()) {
                 print("Total bytes read on main queue: \(totalBytesRead)")
             }
         }
         .response { _, _, _, error in
             if let error = error {
                 print("Failed with error: \(error)")
             } else {
                 print("Downloaded file successfully")
             }
         }
{% endhighlight %}

* HTTP 验证

{% highlight swift %}
let user = "user"
let password = "password"

Alamofire.request(.GET, "https://httpbin.org/basic-auth/\(user)/\(password)")
         .authenticate(user: user, password: password)
         .responseJSON { response in
             debugPrint(response)
         }
{% endhighlight %}

* HTTP 响应状态信息识别

Alamofire 还提供了 HTTP 响应状态的判断识别，通过 validate 方法，对于在我们期望之外的 HTTP 响应状态信息，Alamofire 会提供报错信息：

{% highlight swift %}
Alamofire.request(.GET, "https://httpbin.org/get", parameters: ["foo": "bar"])
         .validate(statusCode: 200..<300)
         .validate(contentType: ["application/json"])
         .response { response in
             print(response)
         }
{% endhighlight %}

validate 方法还提供自动识别机制，我们调用 validate 方法时不传入任何参数，则会自动认为 200...299 的状态吗为正常：

{% highlight swift %}
Alamofire.request(.GET, "https://httpbin.org/get", parameters: ["foo": "bar"])
         .validate()
         .responseJSON { response in
             switch response.result {
             case .Success:
                 print("Validation Successful")
             case .Failure(let error):
                 print(error)
             }
         }
{% endhighlight %}

* 调试状态

我们通过使用 debugPrint 函数，可以打印出请求的详细信息，这样对我们调试非常的方便：

{% highlight swift %}
let request = Alamofire.request(.GET, "https://httpbin.org/get", parameters: ["foo": "bar"])
debugPrint(request)
{% endhighlight %}

这样就会产生如下输出：

{% highlight swift %}
$ curl -i \
    -H "User-Agent: Alamofire" \
    -H "Accept-Encoding: Accept-Encoding: gzip;q=1.0,compress;q=0.5" \
    -H "Accept-Language: en;q=1.0,fr;q=0.9,de;q=0.8,zh-Hans;q=0.7,zh-Hant;q=0.6,ja;q=0.5" \
    "https://httpbin.org/get?foo=bar"
{% endhighlight %}

