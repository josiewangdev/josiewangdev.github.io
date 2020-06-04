---
layout: post
title: "UISearchController的用法"
tags: Project
categories: [Tech]
---

**使用单独的控制器来展示搜索结果**

<br>

使用两个控制器, 一个展示原始数据, 一个来展示搜索的结果;
这其实只是在初始化searchController的时候有些区别, 然后两个控制器分别管理自己的数据源即可

<br>

{% highlight objc %}
 // 创建用于展示搜索结果的控制器
    ResultDisplayController *result = [[ResultDisplayController alloc]init];
    // 创建搜索框
    UISearchController *search = [[UISearchController alloc]initWithSearchResultsController:result];

    self.tableView.tableHeaderView = search.searchBar;
    
    search.searchResultsUpdater = result;
    
    self.searchController = search;
{% endhighlight %}


<br>

{% highlight objc %}
@interface LZResultDisplayController : UIViewController<UISearchResultsUpdating>

@end
{% endhighlight %}

<br>

{% highlight objc %}
#pragma mark - UISearchResultsUpdating
- (void)updateSearchResultsForSearchController:(UISearchController *)searchController {
    
    // 这里处理过滤数据源的逻辑
    NSString *inputStr = searchController.searchBar.text ;
    if (self.results.count > 0) {
        [self.results removeAllObjects];
    }
    for (NSString *str in self.datas) {
        
        if ([str.lowercaseString rangeOfString:inputStr.lowercaseString].location != NSNotFound) {
            
            [self.results addObject:str];
        }
    }
    // 然后刷新表格
    [self.tableView reloadData];
}
{% endhighlight %}

[[Ref.] 系统UISearchController详解](https://www.jianshu.com/p/aa9a153a5b58)

<br>
### Search Scope
<br>

<br>
The iOS UISearchBar makes it dead easy to add a scope bar so users can change what they’re searching through. 
In Interface Builder we can add scope titles of search bar. Also we need to adjust our filter function to use the
 scope. We need to get the current scope from the search bar.
<br>

<br>
The search should also reload when the scope changes, not just when the search text changes so we’ll have to implement the searchDisplayController:shouldReloadTableForSearchScope: fuction:
<br>

{% highlight swift %}
#pragma mark - UISearchResultsUpdating
func searchDisplayController(controller: UISearchDisplayController!, shouldReloadTableForSearchString searchString: String!) -> Bool {
  let selectedIndex = controller.searchBar.selectedScopeButtonIndex
  self.filterContentForSearchText(searchString, scope: selectedIndex)
  return true
}

func searchDisplayController(controller: UISearchDisplayController, shouldReloadTableForSearchScope searchOption: Int) -> Bool {
  let searchString = controller.searchBar.text
  self.filterContentForSearchText(searchString, scope:searchOption)
  return true
}
{% endhighlight %}

[[Ref.] Adding a Search Bar to a Table View in Swift](https://grokswift.com/swift-tableview-search-bar/)

#### Setting Up searchController‘s Parameters
{% highlight swift %}
// 1
searchController.searchResultsUpdater = self
// 2
searchController.obscuresBackgroundDuringPresentation = false
// 3
searchController.searchBar.placeholder = "Search Candies"
// 4
navigationItem.searchController = searchController
// 5
definesPresentationContext = true
{% endhighlight %}


* 1). searchResultsUpdater is a property on UISearchController that conforms to the new protocol, UISearchResultsUpdating. With this protocol, UISearchResultsUpdating will inform your class of any text changes within the UISearchBar.
* 2). By default, UISearchController obscures the view controller containing the information you’re searching. This is useful if you’re using another view controller for your searchResultsController. In this instance, you’ve set the current view to show the results, so you don’t want to obscure your view.
* 3). Here, you set the placeholder to something that’s specific to this app.
* 4). New for iOS 11, you add the searchBar to the navigationItem. This is necessary because Interface Builder is not yet compatible with UISearchController.
* 5). Finally, by setting definesPresentationContext on your view controller to true, you ensure that the search bar doesn’t remain on the screen if the user navigates to another view controller while the UISearchController is active.

[[Ref.] UISearchController Tutorial: Getting Started](https://www.raywenderlich.com/4363809-uisearchcontroller-tutorial-getting-started)
