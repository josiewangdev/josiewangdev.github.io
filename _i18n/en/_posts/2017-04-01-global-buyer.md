---
layout: post
title: Global Buyer
tags: Swift
categories: Projects
cover: images/apps/globalbuyer.jpg
---

## [Global Buyer](https://itunes.apple.com/cn/app/%E7%8E%AF%E7%90%83%E4%B9%B0%E6%89%8B-%E7%9C%9F%E6%AD%A3%E7%9A%84%E6%B4%8B%E8%B4%A7%E6%9D%A5%E8%87%AA%E6%B5%B7%E5%A4%96/id1164383001?mt=8)  

---

参与时间: **2017.04 - 2018.07**

使用语言: **Swift 3**

系统要求: **iOS 9.0 或更高版本**

兼容设备: **iPhone**

![img](/images/apps/globalbuyer.jpg)

---

<br>

### Category

---

#### 1. [主要架构](#main-structure)

1.1 [基本架构](#basic-structure)

1.2 [网络层结构](#network)

---

#### 2. [其他模块](#other-module)

2.1 

2.2 

2.3 

2.4 

2.5 

2.6 

---

#### 3. [3rd Frameworks](#third-party-helpers)

3.1 [MBProgressHUD](#mbprogress)

3.2 [MJRefresh](#mjrefresh)

3.3 [DZNEmptyDataSet](#emptydata)

3.4 [SnapKit](#snapkit)

3.5 [Eureka](/tech/2019-10-30/eureka)

3.6 [Alamofire](/tech/2018-11-09/alamofire)

3.7 [ObjectMapper](#objectmapper)

3.8 [SDWebImage](#sdwebimage)

---

#### 4. [IDE 相关](#IDE)

4.1 [配置不同开发环境](#configuration)

4.2 [推送调试](#debug-notification)

---

#### 5. [遇到的Bug](#bugs)

5.1 [Conflict of DZNEmptyDataSet and MJRefresh](#emptydata-mjrefresh-bug)

5.2 [Update a TableView without animation](#update-tableview)

5.3 [Navigation bar flash to black](#navbar-flash)

5.4 [The issue when set the estimated item size in collection view](#estimated-item-size)

5.5 [Set cell selected status when table view update](#update-cell)

---

#### 6. [参考链接](#ref-links)

---

<br>

### 1. 主要架构 {#main-structure}

---

> #### 1.1  基本架构 {#basic-structure}

在这个项目中, 整体还是使用了传统的 **MVC** 的项目结构。

* M -- Models
    * Enums
    * Custom models

* V -- Views
    * Toast View
    * Alert View
    * Table View Cells
    * Custom Views

* C -- Controllers
    * Common Controllers
    * Custom Controllers

<br>

> #### 1.2  网络层结构 {#network}

---

<br>

> #### 2.1 使用Mapbox实现自定义地图

**iOS Swift MapKit Custom Annotation**

Outline

We will create a point annotation object and assigning a custom image name with the CustomPointAnnotation class.

We will subclass the MKPointAnnotation to set image and assign it on the delegate protocol method viewForAnnotation.

We will add an annotation view to the map after setting the coordinate of the point annotation with a title and a subtitle.

We will implement the viewForAnnotation method which is an MKMapViewDelegate protocol method which gets called for pins to display on the map. viewForAnnotation protocol method is the best place to customise the pin view and assign a custom image to it.

We will dequeue and return a reusable annotation for the given identifier and cast the annotation to our custom CustomPointAnnotation class in order to access the image name of the pin.

We will create a new image set in Assets.xcassets and place image@3x.png and image@2x.png accordingly.

<br>

```swift
CustomPointAnnotation.swift

import UIKit
import MapKit

class CustomPointAnnotation: MKPointAnnotation {
    var pinCustomImageName:String!
}
```

```swift
//  ViewController.swift

import MapKit

class ViewController: UIViewController, MKMapViewDelegate,  CLLocationManagerDelegate {


    @IBOutlet weak var pokemonMap: MKMapView!
    let locationManager = CLLocationManager()
    var pointAnnotation:CustomPointAnnotation!
    var pinAnnotationView:MKPinAnnotationView!

    override func viewDidLoad() {
        super.viewDidLoad()

        //Mark: - Authorization
        locationManager.delegate = self
        locationManager.desiredAccuracy = kCLLocationAccuracyBest
        locationManager.requestAlwaysAuthorization()
        locationManager.startUpdatingLocation()

        pokemonMap.delegate = self
        pokemonMap.mapType = MKMapType.Standard
        pokemonMap.showsUserLocation = true

    }

    func locationManager(manager: CLLocationManager, didUpdateLocations locations: [CLLocation]) {
        let location = CLLocationCoordinate2D(latitude: 35.689949, longitude: 139.697576)
        let center = location
        let region = MKCoordinateRegionMake(center, MKCoordinateSpan(latitudeDelta: 0.025, longitudeDelta: 0.025))
        pokemonMap.setRegion(region, animated: true)

        pointAnnotation = CustomPointAnnotation()
        pointAnnotation.pinCustomImageName = "Pokemon Pin"
        pointAnnotation.coordinate = location
        pointAnnotation.title = "POKéSTOP"
        pointAnnotation.subtitle = "Pick up some Poké Balls"

        pinAnnotationView = MKPinAnnotationView(annotation: pointAnnotation, reuseIdentifier: "pin")
        pokemonMap.addAnnotation(pinAnnotationView.annotation!)
    }

    func locationManager(manager: CLLocationManager, didFailWithError error: NSError) {
        print(error.localizedDescription)
    }

    //MARK: - Custom Annotation
    func mapView(mapView: MKMapView, viewForAnnotation annotation: MKAnnotation) -> MKAnnotationView? {
        let reuseIdentifier = "pin"
        var annotationView = mapView.dequeueReusableAnnotationViewWithIdentifier(reuseIdentifier)

        if annotationView == nil {
            annotationView = MKAnnotationView(annotation: annotation, reuseIdentifier: reuseIdentifier)
            annotationView?.canShowCallout = true
        } else {
            annotationView?.annotation = annotation
        }

        let customPointAnnotation = annotation as! CustomPointAnnotation
        annotationView?.image = UIImage(named: customPointAnnotation.pinCustomImageName)

        return annotationView
    }
}
```
<br>

### 5. 遇到的Bug {#bugs}

---

> #### 5.1 Conflict of DZNEmptyDataSet and MJRefresh {#emptydata-mjrefresh-bug}

**Problem Description:**

If a collection view use both `DZNEmptyDataSet` and `MJRefresh`, The empty data set view will offset the distance of a header.

**Solution:**

Set the content offset to zero in delegate `emptyDataSetWillAppear`.

```objc
- (void)emptyDataSetWillAppear:(UIScrollView *)scrollView {
    scrollView.contentOffset = CGPointZero;
}
```

<br>

> #### 5.2 Update a TableView without animation {#update-tableview}

**Problem Description:**

When update a table view cell and you’re not scrolled at the top (or bottom if you’re inverted) of your table view, but you notice one of the follow behaviors:

* The UITableView scrolls to the top or bottom
* The UITableView looks hung over and flashses back and forth
* The UITableView jiggles just a bit but enough to notice

**Solution:**

1). Do the best you can establishing estimated heights for all of your table cell types. Even if heights are somewhat dynamic this helps the UITableView.

2). Save your scroll position and after updating your TableView and making a call to `endUpdates()` reset the content offset.

Here is the code required to maintain the scroll view’s content offset and prevent unnecessary animations from starting

```swift
let lastScrollOffset = tableView.contentOffset
tableView.beginUpdates()
...
tableView.endUpdates()
tableView.layer.removeAllAnimations()
tableView.setContentOffset(lastScrollOffset, animated: false)
```

<br>

> #### 5.3 Navigation bar flash to black {#navbar-flash}

**Problem Description:**

After setting the navigation bar is hidden, the navigation bar color will flash to black when push it.

**Solution:**

When set navigation bar hidden, set `animated` as `true`.

```swift
navigationController.setNavigationBarHidden(true, animated: true)
```

<br>

> #### 5.4 The issue when set the estimated item size in collection view {#estimated-item-size}


**Problem Description:** 

![collectionview](/images/collectionview_flowlayout/shopping_subtypes.jpg)

Usually when we implement a view like this, we need to use `UICollectionView`, and all the items in collection view should be left-aligned.

You want to use the dynamic width and height of items and align them automatically, so you set the `estimatedItemSize` of the items in collection view. But after that, you will find the crash happens in the customized layout class `UICollectionViewLeftAlignedLayout`.

**System Version:**

Crash happens when the system is lower than `iOS 10.0`.

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

**Solution:**

Use `itemSize` to set the fixed width and height for items rather than use `estimatedItemSize` to set the dynamic ones.

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

> #### 5.5 Set cell selected status when table view update {#update-cell}

If you want to set table view cell selected status in function `cellForRowAtIndexPath`, do not use `setSelected`, but use `selectRowAtIndexPath` instead. Because `setSelected` will be called twice.

![Illustration](/images/20170901/cell_selected.png)

---

<br>

### 6. 参考链接 {#ref-links}

---

* [DZNEmptyDataSet-Github](https://github.com/dzenbot/DZNEmptyDataSet)
* [MJRefresh-Github](https://github.com/CoderMJLee/MJRefresh)
* [UICollectionView section header crashing iOS 8](https://stackoverflow.com/questions/26407149/uicollectionview-section-header-crashing-ios-8)
* [Crash on iOS 8 with auto sizing cells enabled](https://github.com/CSStickyHeaderFlowLayout/CSStickyHeaderFlowLayout/issues/48)
* [Infinite Loop of layoutAttributesForElementsInRect](https://stackoverflow.com/questions/24065014/infinite-loop-of-layoutattributesforelementsinrect)

