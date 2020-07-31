---
layout: post
title: 列表视图
tags: iOS
categories: Dev
---

## 列表

---

#### 1. [列表章节索引](#p1)
#### 2. [动态高度的列表头尾](#p2)
#### 3. [延迟操作至列表重载完成](#p3)
#### 4. [列表项左滑样式](#p4)

---

<br>

> #### 1. 列表章节索引 {#p1}

我们只需要实现以下协议方法, 就可以为列表创建一个索引

```swift
//  返回section的索引标题
func sectionIndexTitles(for tableView: UITableView) -> [String]? {
    return allTitles
}

//  返回每个标题对应的section索引值
func tableView(_ tableView: UITableView, sectionForSectionIndexTitle title: String, at index: Int) -> Int {
    var tpIndex:Int = 0
    allTitles.forEach({
        if $0 == title {
            return tpIndex
        }
        tpIndex += 1
    })
    return 0
}
```

---

<br>

> #### 2. 动态高度的列表头尾 {#p2}

列表视图可以设置表头和表尾，但问题是表头和表尾不能动态布局，必须手动设置一个固定高度。当你的表头表尾需要用动态内容填充的时候，手动设置的这个尺寸可能并不能适配所有的手机型号。

使用以下代码可以根据自动布局的约束重新将表头和表尾调整成合适的大小。

``` swift
func sizeHeaderToFit() {
    if let headerView = tableView.tableHeaderView {
        headerView.setNeedsLayout()
        headerView.layoutIfNeeded()
        let height = headerView.systemLayoutSizeFitting(UILayoutFittingCompressedSize).height
        var frame = headerView.frame
        frame.size.height = height
        headerView.frame = frame
        tableView.tableHeaderView = headerView
    }
}
    
func sizeFooterToFit() {
    if let footerView = tableView.tableFooterView {
        footerView.setNeedsLayout()
        footerView.layoutIfNeeded()
        let height = footerView.systemLayoutSizeFitting(UILayoutFittingCompressedSize).height
        var frame = footerView.frame
        frame.size.height = height
        footerView.frame = frame
        tableView.tableFooterView = footerView
    }
}
```

你只需要在表头表尾填充内容改变的时候调用这些方法。例如

```swift
func configTableViewHeader() {
    //  update the table view header content
    ...
    sizeHeaderToFit()
}
```

---

<br>

> #### 3. 延迟操作至列表重载完成 {#p3}

在实际的项目开发中，经常会有需求等待列表视图刷新或重载完成之后，才能进行下一步操作。

以下两种方法可以解决这个问题：

```objc
[self.tableView reloadData];
[self.tableView layoutIfNeeded];
//  reload finish
//  layoutIfNeeded will force to reload the view and wait the reload finishing
<# write you next step code here #>
```

```objc
[self.tableView reloadData];
dispatch_sync(dispatch_get_main_queue(), ^{
//  reload finish
    <# write you next step code here #>
});
```

---

<br>

> #### 4. 列表项左滑样式 {#p4}

<br>

#### 系统默认左滑样式

<br>

如果只需要使用默认左滑样式，只需要实现 `tableView:commitEditingStyle:forRowAtIndexPath` 数据源方法。

```swift

func tableView(tableView: UITableView, commitEditingStyle editingStyle: UITableViewCellEditingStyle, forRowAtIndexPath indexPath: NSIndexPath) {
    if (editingStyle == UITableViewCellEditingStyleDelete) {
        //  do the remove action
    } else {
        //  handle other other actions
    }
}
```

想要改变系统默认左滑删除选项按钮上的文字，可以用以下方法简单实现

```swift
func tableView(tableView: UITableView, titleForDeleteConfirmationButtonForRowAtIndexPath indexPath:NSIndexPath) -> String { 
    return "自定义删除"; 
}
```

<br>

#### 自定义列表左滑样式

<br>

如果要显示自定义的左滑选项, 必须实现下面的协议方法让列表处于可编辑状态

```swift
func tableView(tableView: UITableView, canEditRowAtIndexPath indexPath: NSIndexPath) -> Bool {
    return true
}
```

同时，需要实现代理方法`tableView:editActionsForRowAtIndexPath`，并在里面创建自定义的`UITableViewRowAction`，当然你也可以定义多个左滑选项

```swift
func tableView(tableView: UITableView, editActionsForRowAtIndexPath indexPath: NSIndexPath) -> [AnyObject]? {
    let more = UITableViewRowAction(style: .Normal, title: "More") { action, index in
        println("more button tapped")
    }
    more.backgroundColor = UIColor.lightGrayColor()
    
    let favorite = UITableViewRowAction(style: .Normal, title: "Favorite") { action, index in
        //  推出编辑模式, 隐藏左滑菜单
        tableView.setEditing(false, animated: true)  
        println("favorite button tapped")
    }
    favorite.backgroundColor = UIColor.orangeColor()
    
    let share = UITableViewRowAction(style: .Normal, title: "Share") { action, index in
        println("share button tapped")
    }
    share.backgroundColor = UIColor.blueColor()
    
    return [share, favorite, more]
}
```

---

<br>

#### 参考链接

* [UITableView左滑删除自定义 - 实现多选项并使用自定义图片](https://www.jianshu.com/p/779f36c21632)
* [Extending UITableViewController](https://spin.atomicobject.com/2017/08/11/swift-extending-uitableviewcontroller/)
* [UIScrollView 实践经验](https://tech.glowing.com/cn/practice-in-uiscrollview/)

