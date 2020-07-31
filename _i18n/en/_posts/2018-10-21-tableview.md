---
layout: post
title: Table View
tags: iOS
categories: Dev
---

## Table View

---

#### 1. [Table view section index](#p1)
#### 2. [Auto sized table view header & footer](#p2)
#### 3. [Delay the execution until table view finish reloading](#p3)
#### 4. [列表中图片加载逻辑的优化](#p4)
#### 5. [Table view cell swipe-left actions](#p5)

---

<br>

> #### 1. Table view section index {#p1}

To implement the section index in table view, just needs simple fuctions as following:

```swift
//  return the index title for each section
func sectionIndexTitles(for tableView: UITableView) -> [String]? {
    return allTitles
}

//  return the coressponding section index for each title
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

> #### 2. Auto sized table view header & footer {#p2}

The table view has a header and footer which allows you to put UI controls above and below the data in your table view. 

The problem is that the header and footer do not participate with Auto Layout.The height of the header and footer need to be set manually. 

You can set an initial size for your header in Xcode, but that may not work for all phone sizes or situations where you have dynamic content in your header that you hide/show at runtime.

The following functions will resize the header or footer to the proper size based on the Auto Layout constraints.

```swift
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

You can call these functions when your header or footer view content has changed.

```swift
func configTableViewHeader() {
    //  update the table view header content
    ...
    sizeHeaderToFit()
}
```

---

<br>

> #### 3. Delay the execution until table view finish reloading {#p3}

In the actual project development, it's a common scenario that we need to wait the table view finishing the reloading or updating then we could do the next step.

There are two solutions:

**1).**
```objc
[self.tableView reloadData];
[self.tableView layoutIfNeeded];

//reload finish
//layoutIfNeeded will force to reload the view and wait the reload finishing
<# write you next step code here #>
```

**2).**
```objc
[self.tableView reloadData];
dispatch_sync(dispatch_get_main_queue(), ^{
//reload finish
<# write you next step code here #>
});
```
---

<br>

#### 4. 列表中图片加载逻辑的优化 {#p4}

1). 当用户手动 drag table view 的时候，会加载 cell 中的图片

2). 在用户快速滑动的减速过程中，不加载过程中 cell 中的图片（但文字信息还是会被加载，只是减少减速过程中的网络开销和图片加载的开销）

3). 在减速结束后，加载所有可见 cell 的图片（如果需要的话）

问题 1：

前面提到，刚开始拖动的时候，dragging 为 YES，decelerating 为 NO；decelerate 过程中，dragging 和 decelerating 都为 YES；decelerate 未结束时开始下一次拖动，dragging 和 decelerating 依然都为 YES。所以无法简单通过 table view 的 dragging 和 decelerating 判断是在用户拖动还是减速过程。

解决这个问题很简单，添加一个变量如 userDragging，在 willBeginDragging 中设为 YES，didEndDragging 中设为 NO。那么 tableView: cellForRowAtIndexPath: 方法中，是否 load 图片的逻辑就是：


```swift
if (!self.userDragging && tableView.decelerating) {  
    cell.imageView.image = nil;
} else {
    //  code for loading image from network or disk
}
```

问题 2：

这么做的话，decelerate 结束后，屏幕上的 cell 都是不带图片的，解决这个问题也不难，你需要一个形如 loadImageForVisibleCells 的方法，加载可见 cell 的图片：

```swift
- (void)loadImageForVisibleCells
{
    NSArray *cells = [self.tableView visibleCells];
    for (GLImageCell *cell in cells) {
        NSIndexPath *indexPath = [self.tableView indexPathForCell:cell];
        [self setupCell:cell withIndexPath:indexPath];
    }
}
```

问题 3：

这个问题可能不容易被发现，在减速过程中如果用户开始新的拖动，当前屏幕的 cell 并不会被加载（前文提到的调用顺序问题导致），而且问题 1 的方案并不能解决问题 3，因为这些 cell 已经在屏上，不会再次经过 cellForRowAtIndexPath 方法。虽然不容易发现，但解决很简单，只需要在 scrollViewWillBeginDragging: 方法里也调用一次 loadImageForVisibleCells 即可。

**再优化**

如果你的 App 只有图片没有文字。配合 SDWebImage，可以再优化了一下这个方法以提升用户体验：

* 如果内存中有图片的缓存，减速过程中也会加载该图片
* 如果图片属于 targetContentOffset 能看到的 cell，正常加载，这样一来，快速滚动的最后一屏出来的的过程中，用户就能看到目标区域的图片逐渐加载
* 你可以尝试用类似 fade in 或者 flip 的效果缓解生硬的突然出现（尤其是像只有图片的 App）

```swift
- (void)scrollViewWillBeginDragging:(UIScrollView *)scrollView
{
    self.targetRect = nil;
    [self loadImageForVisibleCells];
}

- (void)scrollViewWillEndDragging:(UIScrollView *)scrollView withVelocity:(CGPoint)velocity targetContentOffset:(inout CGPoint *)targetContentOffset
{
    CGRect targetRect = CGRectMake(targetContentOffset->x, targetContentOffset->y, scrollView.frame.size.width, scrollView.frame.size.height);
    self.targetRect = [NSValue valueWithCGRect:targetRect];
}

- (void)scrollViewDidEndDecelerating:(UIScrollView *)scrollView
{
    self.targetRect = nil;
    [self loadImageForVisibleCells];
}
```

是否需要加载图片的逻辑：


```
BOOL shouldLoadImage = YES;  
if (self.targetRect && !CGRectIntersectsRect([self.targetRect CGRectValue], cellFrame)) {  
    SDImageCache *cache = [manager imageCache];
    NSString *key = [manager cacheKeyForURL:targetURL];
    if (![cache imageFromMemoryCacheForKey:key]) {
        shouldLoadImage = NO;
    }
}
if (shouldLoadImage) {  
    // load image
}
```

---

<br>

> #### 5. Table view cell swipe-left actions {#p5}

**System default style**

You can use the delegate function `tableView:commitEditingStyle:forRowAtIndexPath` to implement the iOS default swipe actions.

```objc
//ObjC
- (void)tableView:(UITableView *)tableView commitEditingStyle:(UITableViewCellEditingStyle)editingStyle forRowAtIndexPath:(NSIndexPath *)indexPath {
    if (editingStyle == UITableViewCellEditingStyleDelete) {
        // do the remove action
    } else {
        // handle other other actions
    }
}
```

**Customize swipe-left actions**

```swift
func tableView(tableView: UITableView, canEditRowAtIndexPath indexPath: NSIndexPath) -> Bool {
    // 要显示自定义的action,cell必须处于编辑状态
    return true
}
func tableView(tableView: UITableView, commitEditingStyle editingStyle: UITableViewCellEditingStyle, forRowAtIndexPath indexPath: NSIndexPath) {
    // 在此处自定义action
}
```

改变按钮上的文字：

```objc
- (NSString *)tableView:(UITableView *)tableView titleForDeleteConfirmationButtonForRowAtIndexPath:(NSIndexPath *)indexPath
{    
    return @"自定义删除";    
} 
```

#### 自定义图标实现方法

**1). 自定义多个左滑菜单选项**

如果需要超过一个左滑选项，需要实现代理方法tableView:editActionsForRowAtIndexPath，在里面创建多个UITableViewRowAction：

```objc
    // delete action
    UITableViewRowAction *deleteAction = [UITableViewRowAction rowActionWithStyle:UITableViewRowActionStyleNormal title:NSLocalizedString(@"DeleteLabel", @"") handler:^(UITableViewRowAction *action, NSIndexPath *indexPath)
    {
        [tableView setEditing:NO animated:YES];  // 这句很重要，退出编辑模式，隐藏左滑菜单
        [self removeNotificationAction:index];
    }];
    
    // read action
    // 根据cell当前的状态改变选项文字
    NSInteger index = indexPath.section; 
    BOOL isRead = [[NotificationManager instance] read:index];
    NSString *readTitle = isRead ? @"Unread" : @"Read";

    // 创建action
    UITableViewRowAction *readAction = [UITableViewRowAction rowActionWithStyle:UITableViewRowActionStyleNormal title:readTitle handler:^(UITableViewRowAction *action, NSIndexPath *indexPath) 
    {        
        [tableView setEditing:NO animated:YES];  // 这句很重要，退出编辑模式，隐藏左滑菜单
        [[NotificationManager instance] setRead:!isRead index:index];
    }];
    
    //改变按钮背景颜色
    deleteAction.backgroundColor = [UIColor orangeColor];
    readAction.backgroundColor = [UIColor blueColor];
    
    return @[deleteAction, readAction];
```

```swift
func tableView(tableView: UITableView, editActionsForRowAtIndexPath indexPath: NSIndexPath) -> [AnyObject]? {
    let more = UITableViewRowAction(style: .Normal, title: "More") { action, index in
        println("more button tapped")
    }
    more.backgroundColor = UIColor.lightGrayColor()
    
    let favorite = UITableViewRowAction(style: .Normal, title: "Favorite") { action, index in
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

**2). 自定义左滑选项外观**

需要遍历UITableView的子视图拿到选项按钮（UIButton）的reference，因为iOS8-10中，左滑选项是UITableViewCell的子视图，而在iOS11中，左滑选项变成了UITableView的子视图。对iOS8-10和iOS11做不同处理：

```objc
#define SYSTEM_VERSION_GREATER_THAN_OR_EQUAL_TO(v)  ([[[UIDevice currentDevice] systemVersion] compare:v options:NSNumericSearch] != NSOrderedAscending)
#define SYSTEM_VERSION_LESS_THAN(v)                 ([[[UIDevice currentDevice] systemVersion] compare:v options:NSNumericSearch] == NSOrderedAscending)

@property (weak, nonatomic) IBOutlet UITableView *tableView;
@property (strong, nonatomic) NSIndexPath* editingIndexPath;  //当前左滑cell的index，在代理方法中设置

- (void)viewDidLayoutSubviews
{
    [super viewDidLayoutSubviews];
    
    if (self.editingIndexPath)
    {
        [self configSwipeButtons];
    }
}
```

```objc
- (void)configSwipeButtons
{
    // 获取选项按钮的reference
    if (SYSTEM_VERSION_GREATER_THAN_OR_EQUAL_TO(@"11.0"))
    {
        // iOS 11层级 (Xcode 9编译): UITableView -> UISwipeActionPullView
        for (UIView *subview in self.tableView.subviews)
        {
              if ([subview isKindOfClass:NSClassFromString(@"UISwipeActionPullView")] && [subview.subviews count] >= 2)
              {
                   // 和iOS 10的按钮顺序相反
                   UIButton *deleteButton = subsubview.subviews[1];
                   UIButton *readButton = subsubview.subviews[0];
                        
                   [self configDeleteButton:deleteButton];
                   [self configReadButton:readButton];
              }
        }
    }
    else
    {
        // iOS 8-10层级: UITableView -> UITableViewCell -> UITableViewCellDeleteConfirmationView
        NotificationCell *tableCell = [self.tableView cellForRowAtIndexPath:self.editingIndexPath];
        for (UIView *subview in tableCell.subviews)
        {
            if ([subview isKindOfClass:NSClassFromString(@"UITableViewCellDeleteConfirmationView")] && [subview.subviews count] >= 2)
            {
                UIButton *deleteButton = subview.subviews[0];
                UIButton *readButton = subview.subviews[1];
                
                [self configDeleteButton:deleteButton];
                [self configReadButton:readButton];
                [subview setBackgroundColor:[[ColorUtil instance] colorWithHexString:@"E5E8E8"]];
            }
        }
    }
    
    [self configDeleteButton:deleteButton];
    [self configReadButton:readButton];
}
```

```objc
- (void)tableView:(UITableView *)tableView willBeginEditingRowAtIndexPath:(NSIndexPath *)indexPath
{
    self.editingIndexPath = indexPath;
    [self.view setNeedsLayout];   // 触发-(void)viewDidLayoutSubviews
}

- (void)tableView:(UITableView *)tableView didEndEditingRowAtIndexPath:(NSIndexPath *)indexPath
{
    self.editingIndexPath = nil;
}
```

```objc
- (void)configDeleteButton:(UIButton*)deleteButton
{
    if (deleteButton)
    {
        [deleteButton.titleLabel setFont:[UIFont fontWithName:@"SFUIText-Regular" size:12.0]];
        [deleteButton setTitleColor:[[ColorUtil instance] colorWithHexString:@"D0021B"] forState:UIControlStateNormal];
        [deleteButton setImage:[UIImage imageNamed:@"Delete_icon_.png"] forState:UIControlStateNormal];
        [deleteButton setBackgroundColor:[[ColorUtil instance] colorWithHexString:@"E5E8E8"]];
        // 调整按钮上图片和文字的相对位置（该方法的实现在下面）
        [self centerImageAndTextOnButton:deleteButton]; 
    }
}

- (void)configReadButton:(UIButton*)readButton
{
    if (readButton)
    {
        [readButton.titleLabel setFont:[UIFont fontWithName:@"SFUIText-Regular" size:12.0]];
        [readButton setTitleColor:[[ColorUtil instance] colorWithHexString:@"4A90E2"] forState:UIControlStateNormal];
        // 根据当前状态选择不同图片
        BOOL isRead = [[NotificationManager instance] read:self.editingIndexPath.row];
        UIImage *readButtonImage = [UIImage imageNamed: isRead ? @"Mark_as_unread_icon_.png" : @"Mark_as_read_icon_.png"];
        [readButton setImage:readButtonImage forState:UIControlStateNormal];

        [readButton setBackgroundColor:[[ColorUtil instance] colorWithHexString:@"E5E8E8"]];
        // 调整按钮上图片和文字的相对位置（该方法的实现在下面）
        [self centerImageAndTextOnButton:readButton];
    }
}

- (void)centerImageAndTextOnButton:(UIButton*)button
{
    // this is to center the image and text on button.
    // the space between the image and text
    CGFloat spacing = 35.0;
    
    // lower the text and push it left so it appears centered below the image
    CGSize imageSize = button.imageView.image.size;
    button.titleEdgeInsets = UIEdgeInsetsMake(0.0, - imageSize.width, - (imageSize.height + spacing), 0.0);
    
    // raise the image and push it right so it appears centered above the text
    CGSize titleSize = [button.titleLabel.text sizeWithAttributes:@{NSFontAttributeName: button.titleLabel.font}];
    button.imageEdgeInsets = UIEdgeInsetsMake(-(titleSize.height + spacing), 0.0, 0.0, - titleSize.width);
    
    // increase the content height to avoid clipping
    CGFloat edgeOffset = (titleSize.height - imageSize.height) / 2.0;
    button.contentEdgeInsets = UIEdgeInsetsMake(edgeOffset, 0.0, edgeOffset, 0.0);
    
    // move whole button down, apple placed the button too high in iOS 10
    if (SYSTEM_VERSION_LESS_THAN(@"11.0"))
    {
        CGRect btnFrame = button.frame;
        btnFrame.origin.y = 18;
        button.frame = btnFrame;
    }
}
```

---

<br>
#### Reference links

* [UITableView左滑删除自定义 - 实现多选项并使用自定义图片](https://www.jianshu.com/p/779f36c21632)
* [Extending UITableViewController](https://spin.atomicobject.com/2017/08/11/swift-extending-uitableviewcontroller/)
