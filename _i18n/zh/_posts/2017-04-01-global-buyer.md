---
layout: post
title: 环球买手
tags: Swift
categories: Projects
cover: https://coderjosie-1258689192.cos.ap-nanjing.myqcloud.com/projects/global_buyer/globalbuyer.jpg
description: 2017.04 - 2018.07
---

## [环球买手](https://itunes.apple.com/cn/app/%E7%8E%AF%E7%90%83%E4%B9%B0%E6%89%8B-%E7%9C%9F%E6%AD%A3%E7%9A%84%E6%B4%8B%E8%B4%A7%E6%9D%A5%E8%87%AA%E6%B5%B7%E5%A4%96/id1164383001?mt=8)  

---

![img](https://coderjosie-1258689192.cos.ap-nanjing.myqcloud.com/projects/global_buyer/globalbuyer.jpg)

---

<br>

### 目录

---

#### 1. [概述](#p1)

1.1 [基本信息](#p11)

1.2 [结构解读](#p12)

1.3 [第三方框架的使用](#p13)

---

#### 2. [开发随笔](#p2)

2.1 [使用Mapbox实现自定义地图](#mapbox)

2.2 [UIScrollView多层嵌套中的联动效果实现](#multi_scrollview)

2.3 [DZNEmptyDataSet与MJRefresh的冲突](#emptydata-mjrefresh-bug)

2.4 [无动画效果更新TableView内容](#update-tableview)

2.5 [导航栏闪变黑色](#navbar-flash)

2.6 [集合视图设置预估尺寸的问题](#estimated-item-size)

2.7 [在列表内刷新cell的选中状态](#update-cell)

2.8 [项目本地化](#i18n)

---

#### 3. [参考链接](#ref-links)

---

<br>

### 1. 概述 {#p1}

---

环球买手是我在墨尔本工作旅行时加入一个华人老板公司时做的产品。 切合了很多海外华人的需求, 即做代购, 当买手, 为国内的消费者代购澳洲产品, 例如奶粉, 保健品, 常用药品之类。 

整个 App 功能包括展示海外买手位置, 买卖代购商品, 整个订单由下单到退换货等完整流程。 

这个产品应该是我在产品设计与需求确定相关方面投入精力最多的一款 App。 一方面是本身对产品UI设计有极大兴趣, 另一方面是公司人员不多, 每个人由主观能动性能够发挥的余地也很多, 有时我也会亲自上阵参与设计, 修改UI, 自发增加一些动画效果之类的开发等等。

在这个项目推进的过程中, 真正体会到了初创 start up 一类公司的初期项目的发展过程, 从仅有的一个想法到一个成熟产品的上线, 虽没有取得光辉的成果, 但过程仍旧值得记录与纪念。
 
---

<br>

> #### 1.1 基本信息 {#p11}

使用语言: **Swift 3**

系统要求: **iOS**

参与时间: **2017.04 - 2018.07**

负责部分: **个人独立开发所有模块**

<br>

> #### 1.2 结构解读 {#p12}

#### 主要框架

* M -- Models
    * Common Models

* V -- Views
    * Common Views (toast View, alert View, cells, etc.)

* Modules
    * Module 1
        * Module 1's models
        * Module 1's views
        * Module 1's controllers
    * Module 2
        * Module 2's models
        * Module 2's views
        * Module 2's controllers
    * Module 3
        * Module 3's models
        * Module 3's views
        * Module 3's controllers

        ... ...

<br>

#### 网络层结构

![img](https://coderjosie-1258689192.cos.ap-nanjing.myqcloud.com/projects/global_buyer/p12.jpg)

<br>

以 **ShippingCompany** 业务模块举例

```swift
//
//  NetworkBaseUtility.swift
//

import Alamofire

typealias SuccessCalllback = (_ result: [String: Any]?)->Void
typealias FailureCallback = (_ msg: String?)->Void
typealias SuccessWithCodeCalllback = (_ code:Int,_ result: [String:AnyObject]?,_ msg: String?)->Void
typealias FailureWithCodeCallback = (_ code:Int, _ msg: String?)->Void
typealias VoidCallback = ()->Void

class NetworkBaseUtility {

    static fileprivate var headers : HTTPHeaders {
        get {
            if UserUtility.getAccessToken() != nil {
                return [
                    "Authorization":"Bearer \(UserUtility.getAccessToken()!)",
                    "Content-Type": "application/json"
                ]
            }
            return ["Content-Type": "application/json"]
        }
    }
    
    static let afManager = NetworkBaseUtility.defaultAlamofireSessionManager()
    
    fileprivate class func alamofireSessionManager(withTimeout isLongTimeOut:Bool, hasAuthorizationHeader: Bool) -> SessionManager{
        
        let configuration = URLSessionConfiguration.default
        
        if !isLongTimeOut {
            configuration.timeoutIntervalForRequest = 5 // seconds
            configuration.timeoutIntervalForResource = 5
        }
        
        let manager = Alamofire.SessionManager(configuration: configuration)
        return manager
        
    }
    
    private class func defaultAlamofireSessionManager() -> SessionManager{
        return alamofireSessionManager(withTimeout: true, hasAuthorizationHeader: true)
    }
    
    fileprivate class func handleResponseResult(withResult result: [String:AnyObject]?, success: SuccessCalllback, failure: FailureCallback) {
        if result != nil {
            if result!["code"] as! Int == 0 {
                success(result!["data"] as? Dictionary)
            }
            else{
                failure(result!["message"] as? String)
            }
        }
    }
    
    fileprivate class func handleResponseResultWithCode(withResult result: [String:AnyObject]?, success: SuccessWithCodeCalllback, failure: FailureWithCodeCallback) {
        if result != nil {
            if result!["code"] as! Int == 0 || result!["code"] as! Int == 10 {
                success(result!["code"] as! Int, result!["data"] as? Dictionary, result!["message"] as? String)
            }
            else{
                failure(result!["code"] as! Int, result!["message"] as? String)
            }
        }
    }
    
    fileprivate class func getVersioningURL(url: String) -> String {
        return (url + URL_VERSION_SUFFIX)
    }
    
    class func networkGetRequest(withURL url: String, parameters: [String:Any]?, success: @escaping SuccessCalllback, failure: @escaping FailureCallback) {
        
        let paraSend = (parameters?.count == 0 ? nil : parameters)
        
        afManager.request(getVersioningURL(url: url), method: .get, parameters: paraSend, encoding: JSONEncoding.default, headers: headers)
            .validate()
            .responseJSON{ response in
                switch response.result {
                case .success(let value):
                    handleResponseResult(withResult: value as? Dictionary, success: success, failure: failure)
                case .failure(let error):
                    failure(error.localizedDescription)
                }
        }
    }
    
    class func networkPostRequest(withURL url: String, parameters: [String: Any]?, success: @escaping SuccessCalllback, failure: @escaping FailureCallback) {
        
        let paraSend = (parameters?.count == 0 ? nil : parameters)
        
        afManager.request(getVersioningURL(url: url), method: .post, parameters: paraSend, encoding: JSONEncoding.default, headers: headers)
            .validate()
            .responseJSON{ response in
                switch response.result {
                case .success(let value):
                    handleResponseResult(withResult: value as? Dictionary, success: success, failure: failure)
                case .failure(let error):
                    failure(error.localizedDescription)
                }
        }
    }
    
    class func networkDeleteRequest(withURL url: String, parameters: [String:Any]?, success: @escaping SuccessCalllback, failure: @escaping FailureCallback) {
        
        let paraSend = (parameters?.count == 0 ? nil : parameters)
        
        afManager.request(getVersioningURL(url: url), method: .delete, parameters: paraSend, encoding: JSONEncoding.default, headers: headers)
            .validate()
            .responseJSON{ response in
                switch response.result {
                case .success(let value):
                    handleResponseResult(withResult: value as? Dictionary, success: success, failure: failure)
                case .failure(let error):
                    failure(error.localizedDescription)
                }
        }
    }
    
    class func networkGetRequestWithCode(withURL url: String, parameters: [String:AnyObject]?, success: @escaping SuccessWithCodeCalllback, failure: @escaping FailureWithCodeCallback) {
        
        let paraSend = (parameters?.count == 0 ? nil : parameters)

        afManager.request(getVersioningURL(url: url), method: .get, parameters: paraSend, encoding: JSONEncoding.default, headers: headers)
            .validate()
            .responseJSON{ response in
            switch response.result {
            case .success:
                let valueDic : [String : AnyObject] = response.result.value as! Dictionary
                handleResponseResultWithCode(withResult: valueDic, success: success, failure: failure)
            case .failure(let error):
                failure(NETWORK_FAILURE_CODE, error.localizedDescription)
            }
        }
    }
    
    class func networkPostRequestWithCode(withURL url: String, parameters: [String:Any]?, success: @escaping SuccessWithCodeCalllback, failure: @escaping FailureWithCodeCallback) {
        
        let paraSend = (parameters?.count == 0 ? nil : parameters)

        afManager.request(getVersioningURL(url: url), method: .post, parameters: paraSend, encoding: JSONEncoding.default, headers: headers)
            .validate()
            .responseJSON{ response in
            switch response.result {
            case .success:
                let valueDic : [String : AnyObject] = response.result.value as! Dictionary
                handleResponseResultWithCode(withResult: valueDic, success: success, failure: failure)
            case .failure(let error):
                failure(NETWORK_FAILURE_CODE, error.localizedDescription)
            }
        }
    }
    
    class func networkDeleteRequestWithCode(withURL url: String, parameters: [String:AnyObject]?, success: @escaping SuccessWithCodeCalllback, failure: @escaping FailureWithCodeCallback) {
        
        let paraSend = (parameters?.count == 0 ? nil : parameters)
        
        afManager.request(getVersioningURL(url: url), method: .delete, parameters: paraSend, encoding: JSONEncoding.default, headers: headers)
            .validate()
            .responseJSON{ response in
                switch response.result {
                case .success:
                    let valueDic : [String : AnyObject] = response.result.value as! Dictionary
                    handleResponseResultWithCode(withResult: valueDic, success: success, failure: failure)
                case .failure(let error):
                    failure(NETWORK_FAILURE_CODE, error.localizedDescription)
                }
        }
    }
    
}
```

```swift
//
//  ShippingCompanyUtility.swift
//

import ObjectMapper

class ShippingCompanyUtility {
    class func listShippingCompanies(_ inAreaIds:String,callback:@escaping (NSError?,[ShippingCompany]?)->Void){
        let url = "\(PublicUtility.baseApiUrl)/shippingCompany/list"
        let parameters: Dictionary<String,AnyObject> = [
            "in_area_ids":inAreaIds as AnyObject
        ]
        
        NetworkBaseUtility.networkPostRequestWithCode(withURL: url, parameters: parameters, success: { (code, result, message) in            
            let viewModel = Mapper<ShippingCompany>().mapArray(JSONArray: result!["shipping_companies"] as! [[String : Any]])
            callback(nil,viewModel)
        }) { (code, message) in
            callback(NSError.errorFromResponse(code: code, message: message!),nil)
        }
    }
}
```

```swift
ShippingCompanyUtility.listShippingCompanies("014806553810941941", callback: { (error, shippingCompanies) in
    if weakSelf != nil {
        PublicUtility.hideDefaultHUDViewInView(view: weakSelf!.view)
    }
    if error == nil {
        let vc = ShippingCompanySelectionTableViewController()
        vc.shippingCompanies = shippingCompanies!
        vc.shippingCompanySelectionDelegate = self
        weakSelf?.navigationController?.pushViewController(vc, animated: true)
    }else{
        AlertViewUtility.showWarningView(error!.localizedDescription)
    }
})

```

<br>

> #### 1.3 第三方框架的使用 {#p13}

* [ChatKit - LeanCloud 轻量级 IM 聊天组件]()
* [Mapbox-iOS-SDK - 自定义地图]()
* [BarrageRenderer - 弹幕渲染器]()
* [DZNEmptyDataSet - 列表空数据时的效果处理]()
* [DZNPhotoPickerController - 相册图片选择、剪裁编辑器]()
* [MBProgressHUD - 创建prompt]()
* [MJRefresh - 列表上拉刷新，下拉加载]()
* [SnapKit - 使用纯代码方案解决屏幕大小自适应问题]()
* [Eureka - 创建动态表单的框架]()
* [Alamofire - 网络层]()
* [SDWebImage - 网络加载图片]()
* [ObjectMapper - 轻量级的模型与字典、数组等数据结构的转换]()
* [Bugly - 崩溃监测统计]()

---

<br>

### 2. 开发随笔 {#p2}

---

<br>

> #### 2.1 使用Mapbox实现自定义地图 {#mapbox}

<br>

#### 基于Mapbox的FBAnnotationClustering库使用实现聚合效果的方法

<br>

Mapbox可以只需要在Mapbox的Studio后台中编辑修改你的地图样式，无需更新前端线上已有app，App内的地图的UI就能动态更新。

但因Mapbox使用的js语言与源生应用语言不同，以及它实现我们需要的聚合效果并不能满足我们的项目需求。

在该项目中，有如下图所示设计需求: 地图上需要有多个Pin坐标，并且这些坐标是可以有聚合（Cluster）效果的，并且聚合时点击显示的数据展示页面与散开时点击的数据展示页面是不同的UI效果。

<br>

![img](https://coderjosie-1258689192.cos.ap-nanjing.myqcloud.com/projects/global_buyer/mapbox.gif)

<br>

首先, 能够直接使用的就是Mapbox原生API实现聚合效果的方法。

这样的实现方法没有问题，但是在获取Cluter聚合的具体数据时，便不能满足我们的需求了。因为它的API并没有能够让我们去解析每个聚合点内的具体数据到底是什么, 从而我没有办法展示下一页面的列表信息。

为解决这个问题，我使用了FBAnnotationClustering这个第三方开源库，该库实现了在苹果自带系统地图基础上的Cluster聚合数据效果。

当然，这个库支持的数据类型是iOS源生Mapkit所持有的数据类型，当在使用这个库的时候，还需将它改写成适用于Mapbox所持有的数据类型。通过这个库的支持，用如下代码实现了最后我们UI所需要达到的效果。

```swift
fileprivate func setupMapViewUI() {

    var annotations : [MGLPointFeature] = []
    for buyer in buyers {
        let annotation = MGLPointFeature()
        annotation.coordinate = CLLocationCoordinate2DMake(buyer.lastLatitude, buyer.lastLongitude)
        annotation.title = buyer.displayName
        annotations.append(annotation)
    }

    let source = MGLShapeSource(identifier: sourceReuseId, features: annotations, options: [.clustered: true, .clusterRadius: 30])
    mapView.style?.addSource(source)

    let iconImage = UIImage(named: "pinblue")
    mapView.style?.setImage(iconImage!.withRenderingMode(.alwaysOriginal), forName: "icon")

    let clusteredPinLayer = MGLSymbolStyleLayer(identifier: clusteredLayerReuseId, source: source)
    clusteredPinLayer.iconImageName = MGLStyleValue(rawValue: "icon")
    clusteredPinLayer.textColor = MGLStyleValue(rawValue: UIColor.white)
    clusteredPinLayer.textFontSize = MGLStyleValue(rawValue: NSNumber(value: 12))
    clusteredPinLayer.text = MGLStyleValue(rawValue: "{point_count}")
    clusteredPinLayer.predicate = NSPredicate(format: "%K == YES", "cluster")
    clusteredPinLayer.textOffset = MGLStyleValue(rawValue: NSValue.init(cgVector: CGVector(dx: 0, dy: -0.2)))
    mapView.style?.addLayer(clusteredPinLayer)

    let unclusteredPinLayer = MGLSymbolStyleLayer(identifier: unclusteredLayerReuseId, source: source)
    unclusteredPinLayer.iconImageName = MGLStyleValue(rawValue: "icon")
    unclusteredPinLayer.textColor = MGLStyleValue(rawValue: UIColor.white)
    unclusteredPinLayer.textFontSize = MGLStyleValue(rawValue: NSNumber(value: 12))
    unclusteredPinLayer.text = MGLStyleValue(rawValue: "1")
    unclusteredPinLayer.predicate = NSPredicate(format: "%K != YES", "cluster")
    unclusteredPinLayer.textOffset = MGLStyleValue(rawValue: NSValue.init(cgVector: CGVector(dx: 0, dy: -0.2)))
    mapView.style?.addLayer(unclusteredPinLayer)

}
```

```swift
//MARK: - MGLMapViewDelegate

func mapView(_ mapView: MGLMapView, viewFor annotation: MGLAnnotation) -> MGLAnnotationView? {

    var reuseId = ""

    if !annotation.isKind(of: GBBuyerPointAnnotation.self)  {

        reuseId = "Cluster"

        var clusterView = mapView.dequeueReusableAnnotationView(withIdentifier: reuseId)

        if clusterView == nil {
            clusterView = MGLAnnotationView(annotation: annotation, reuseIdentifier: reuseId)
        }else {
            clusterView?.annotation = annotation
        }

        for subview in (clusterView?.subviews)! {
            subview.removeFromSuperview()
        }

        let clusterAnnotations = annotation as! FBAnnotationCluster
        let firstAnnotation = clusterAnnotations.annotations.first as! GBBuyerPointAnnotation
        let clusterCountView = firstAnnotation.getClusteringView(clusterAnnotations.annotations.count)
        clusterView?.frame = CGRect(origin: (clusterView?.frame.origin)!, size: clusterCountView.frame.size)
        clusterView?.centerOffset = CGVector(dx: 0, dy: -clusterCountView.frame.size.height / 2.0)
        clusterView?.addSubview(clusterCountView)

        return clusterView

    } else {

        reuseId = "Pin"

        var pinView = mapView.dequeueReusableAnnotationView(withIdentifier: reuseId)

        if pinView == nil {
            pinView = MGLAnnotationView(annotation: annotation, reuseIdentifier: reuseId)
        }else {
            pinView?.annotation = annotation
        }

        let pinAnnotation = annotation as! GBBuyerPointAnnotation
        let pinImageView = pinAnnotation.getPinView()

        pinView?.frame = CGRect(origin: (pinView?.frame.origin)!, size: pinImageView.frame.size)
        pinView?.centerOffset = CGVector(dx: 0, dy: -pinImageView.frame.size.height / 2.0)
        pinView?.addSubview(pinImageView)

        return pinView

    }

}
```

```swift
fileprivate let clusteringManager = FBClusteringManager()

fileprivate func updateBuyerAnnotationsOnMapView(_ mapView: MGLMapView) {

    weak var weakSelf = self

    OperationQueue().addOperation ({

        let leftBottomPoint = MKMapPointForCoordinate(mapView.visibleCoordinateBounds.sw)
        let rightTopPoint = MKMapPointForCoordinate(mapView.visibleCoordinateBounds.ne)
        let visibleRectWidth = rightTopPoint.x - leftBottomPoint.x
        let scale = Double(mapView.bounds.size.width) / visibleRectWidth

        let visibleMapRect = MKMapRect(origin: MKMapPoint(x: leftBottomPoint.x, y: rightTopPoint.y), size: MKMapSize(width: visibleRectWidth, height: leftBottomPoint.y - rightTopPoint.y))

        let annotations = weakSelf?.clusteringManager.clusteredAnnotationsWithinMapRect(visibleMapRect, withZoomScale: scale)

        weakSelf?.clusteringManager.displayAnnotations(annotations!, onMapView: mapView)

    })

}
```

<br>

> #### 2.2 [UIScrollView多层嵌套中的联动效果实现](#multi_scrollview)

在项目中，经常会出现多层滚动视图嵌套的才能实现的UI需求。但是多层滚动视图的嵌套，通常会引发手势不连贯，用户操作滑动不顺畅等问题。因为通常情况下，只有最上层滚动视图能够响应手势动作。

如果想让ScrollView的滑动效果能够平滑的由上层穿透到下层，无缝滚动连接，则需要将下层视图的手势穿透响应打开。在这里，我把TableView与CollectionView都算作ScrollView的一种。

例如我现在所做的项目中，有如下UI设计需求，只能由多层的ScrollView嵌套去实现。

<br>

![img](https://coderjosie-1258689192.cos.ap-nanjing.myqcloud.com/projects/global_buyer/view_hierarchy.jpg) | ![img](https://coderjosie-1258689192.cos.ap-nanjing.myqcloud.com/projects/global_buyer/multi_scrollview.gif)

<br>

**1).** 先实现UIGestureRecognizerDelegate协议的方法，UIScrollView及其子类已经实现了该方法，只需将返回值返回为YES/true

```objc
//ObjC
- (BOOL)gestureRecognizer:(UIGestureRecognizer *)gestureRecognizer shouldRecognizeSimultaneouslyWithGestureRecognizer:(UIGestureRecognizer *)otherGestureRecognizer {
    return YES;
}
```

```swift
//Swift3
func gestureRecognizer(_ gestureRecognizer: UIGestureRecognizer, shouldRecognizeSimultaneouslyWith otherGestureRecognizer: UIGestureRecognizer) -> Bool {
    return true
}
```

```objc
//Objective-C
- (BOOL)gestureRecognizer:(UIPanGestureRecognizer *)gestureRecognizer shouldRecognizeSimultaneouslyWithGestureRecognizer:(UISwipeGestureRecognizer *)otherGestureRecognizer 
{
    return YES;
}
```

<br>

**2).** 判断何时不让某个scrollview改变偏移量。通过设置子滚动视图与主滚动视图的代理协议方法，确定了他们之间滚动时偏移量的关系后，平滑的多ScrollView滚动效果就实现了。

```swift
//该项目是用Swift3实现的，在这里只贴一下Swift的实现代码

fileprivate enum ScrollViewOffsetType {
    case max
    case min
    case center
}

//MainScrollView的UIScrollViewDelegate协议方法

func scrollViewDidScroll(_ scrollView: UIScrollView) {

    if scrollView.contentOffset.y + scrollView.bounds.height >= scrollView.contentSize.height {
        mainScrollViewOffsetType = .max
    }else if scrollView.contentOffset.y <= 0 {
        mainScrollViewOffsetType = .min
    }else {
        mainScrollViewOffsetType = .center
    }

    //SlideTabView即为上图所示的SubScrollView，是我写的一个嵌套子滑动视图的一个通用控件
    if slideTabView.currentIndex == 0 && productCollectionViewOffsetType == .center {
        scrollView.contentOffset = CGPoint(x: scrollView.contentOffset.x, y: scrollView.contentSize.height - scrollView.bounds.height)
    }

    if slideTabView.currentIndex == 1 && buyerCommentTableViewOffsetType == .center {
        scrollView.contentOffset = CGPoint(x: scrollView.contentOffset.x, y: scrollView.contentSize.height - scrollView.bounds.height)
    }
}

//以SubScrollView中的子滚动视图CollectionView为例，当其滚动到顶部，开启MainScrollView的滚动，在这里ProductCollectionView也是我写的一个通用组件，并且我将它的UIScrollViewDelegate代理协议方法通过block函数回调实现

weak var weakSelf = self
productCollectionView.handleCollectionViewDidScroll = { (scrollView) in

    if scrollView.contentOffset.y <= 0 {
        weakSelf?.productCollectionViewOffsetType = .min
    } else {
        weakSelf?.productCollectionViewOffsetType = .center
    }

    if weakSelf?.mainScrollViewOffsetType == .min {
        scrollView.contentOffset = CGPoint.zero
    }

    if weakSelf?.mainScrollViewOffsetType == .center {
        scrollView.contentOffset = CGPoint.zero
    }

}

```

<br>

> #### 2.3 DZNEmptyDataSet与MJRefresh的冲突 {#emptydata-mjrefresh-bug}

**Bug描述:**

在collection view中同时使用 `DZNEmptyDataSet` 和 `MJRefresh`时，`MJRefresh`会造成空数据页面错位一个header的高度。

**解决方法:**

在 `emptyDataSetWillAppear` 代理方法中将滚动视图的偏移量归零。

```objc
- (void)emptyDataSetWillAppear:(UIScrollView *)scrollView {
    scrollView.contentOffset = CGPointZero;
}
```

<br>

> #### 2.4 无动画效果更新TableView内容 {#update-tableview}

**问题描述:**

当更新一个table view cell的内容并且没有滚动当前列表到最顶（或最底），会发现有以下几种情况出现：

* UITableView 滚动到了最顶或最底部
* UITableView 来回大幅闪动
* UITableView 小幅度闪烁了一下但是能够看出

**解决方法:**

1). 尽最大可能的估计你使用的所有类型的 table cell的大约高度，即使有些地方的table cell的高度使用的是动态高度。

2). 暂存列表的滚动位置，在完成TableView的更新之后，调用 `endUpdates()` 重置滚动视图的内容偏移值。

以下是无动画重置滚动视图内容偏移量的代码

```swift
let lastScrollOffset = tableView.contentOffset
tableView.beginUpdates()
...
tableView.endUpdates()
tableView.layer.removeAllAnimations()
tableView.setContentOffset(lastScrollOffset, animated: false)
```

<br>

> #### 2.5 导航栏闪变黑色 {#navbar-flash}

**问题描述:**

设置Navigation Bar隐藏后，再push时发现导航栏会闪变黑色。

**解决方法:**

在设置导航栏隐藏时，将 `animated` 值设为 `true` .

```swift
navigationController.setNavigationBarHidden(true, animated: true)
```

<br>

> #### 2.6 集合视图设置预估尺寸的问题 {#estimated-item-size}

**问题描述:**

![collectionview](https://coderjosie-1258689192.cos.ap-nanjing.myqcloud.com/projects/global_buyer/shopping_subtypes.jpg)

通常在写这样的一个页面中，会需要使用到 `UICollectionView` 并且需要其中的 items 都要左对齐。

因为想要使其使用动态高度和宽度自动布局，将 collection view 里的 item 设置 `estimatedItemSize` 后，则会在自定义左对齐布局类 `UICollectionViewLeftAlignedLayout` 中，发生崩溃。

**系统版本:**

崩溃只有当系统版本低于 `iOS 10.0` 的时候才会发生。

```swift
//  LeftAlignedLayout.class
//  LeftAlignedLayout inherits from UICollectionViewFlowLayout

override func layoutAttributesForElements(in rect: CGRect) -> [UICollectionViewLayoutAttributes]? {
    var attrsCopy = [UICollectionViewLayoutAttributes]()
    //crash here
    if let attrs = super.layoutAttributesForElements(in: rect) { 
        attrs.forEach({
            attrsCopy.append($0.copy() as! UICollectionViewLayoutAttributes)
        })
    }
    for attribute in attrsCopy {
        if attribute.representedElementKind == nil {
            if let attr = layoutAttributesForItem(at: attribute.indexPath) {
                attribute.frame = attr.frame
            }
        }
    }
    return attrCopy
}
```

**解决方法:**

不再使用动态布局的 `estimatedItemSize` 而是使用固定大小 `itemSize`。

```swift
let operatingSystemVersion = OperatingSystemVersion(majorVersion: 10, minorVersion: 0, patchVersion: 0)
if ProcessInfo().isOperatingSystemAtLeast(operatingSystemVersion) {
    flowLayout.estimatedItemSize = CGSize(width: 100, height: 20)
}else {
    //system version is lower than 10.0.0
    flowLayout.itemSize = CGSize(width: 100, height: 20)
}
```

<br>

> #### 2.7 在列表内刷新cell的选中状态 {#update-cell}

如果想要在 `cellForRowAtIndexPath` 方法中设置cell的选中状态，不要使用 `setSelected` 方法， 而是使用 `selectRowAtIndexPath` 方法，因为 `setSelected` 会被调起两遍。

![Illustration](https://coderjosie-1258689192.cos.ap-nanjing.myqcloud.com/projects/global_buyer/cell_selected.png)

<br>

> #### 2.8 项目本地化 {#i18n}

#### 本地化字符串

**1). PROJECT -> Info -> Localizations -> 添加其他语言**

![Step1](https://coderjosie-1258689192.cos.ap-nanjing.myqcloud.com/projects/global_buyer/localized_step1.png) 

<br>

**2).** 在工程中，选择 **Resource** 下的 **Strings File** 文件类型，创建新文件，命名为 **Localizable.strings** 。

![Step2](https://coderjosie-1258689192.cos.ap-nanjing.myqcloud.com/projects/global_buyer/localized_step2.png) 

<br>

**3).** 选中创建的 **Localizable.strings** 文件，在右侧属性面板中点击 **Localize...** 创建本地化资源。

![Step3](https://coderjosie-1258689192.cos.ap-nanjing.myqcloud.com/projects/global_buyer/localized_step3.png) 

<br>

**4).** 分别在不同语言文件下，用相同的 **key** 值配置不同语言的显示值。

```swift
//  Localizable.strings(English)
"key" = "Welcome";
```

```swift
//  Localizable.strings(Simplified)
"key" = "欢迎";
```

<br>

**5).** 使用 **NSLocalizedString** 和配置的 **key** 来显示字符串。

```swift
NSLocalizedString("key", value: "", comment: "")
```

<br>

#### 本地化应用名称

只需在不同语言的 **.strings** 文件中，像上述方法一样使用 **CFBundleDisplayName** 作为键设置不同的显示值，之后在 **info.plist**文件中使用 **CFBundleDisplayName** 即可。

```swift
//  Localizable.strings(English)
"CFBundleDisplayName" = "English App";
```

```swift
//  Localizable.strings(Simplified)
"CFBundleDisplayName" = "中文应用";
```

---

<br>

### 3. 参考链接 {#ref-links}

---

* [FBAnnotationClustering](https://github.com/infinum/FBAnnotationClustering)
* [FBAnnotationClusteringSwift](https://github.com/ribl/FBAnnotationClusteringSwift)
* [StackOverflow - Swift MapKit Custom Annotation](https://stackoverflow.com/questions/38274115/ios-swift-mapkit-custom-annotation)
* [iOS解决方案：多个scrollview联动](http://www.jianshu.com/p/42479a0e8ac6)
* [DZNEmptyDataSet-Github](https://github.com/dzenbot/DZNEmptyDataSet)
* [MJRefresh-Github](https://github.com/CoderMJLee/MJRefresh)
* [UICollectionView section header crashing iOS 8](https://stackoverflow.com/questions/26407149/uicollectionview-section-header-crashing-ios-8)
* [Crash on iOS 8 with auto sizing cells enabled](https://github.com/CSStickyHeaderFlowLayout/CSStickyHeaderFlowLayout/issues/48)
* [Infinite Loop of layoutAttributesForElementsInRect](https://stackoverflow.com/questions/24065014/infinite-loop-of-layoutattributesforelementsinrect)
