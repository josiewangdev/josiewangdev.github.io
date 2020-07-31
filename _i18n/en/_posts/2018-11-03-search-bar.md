---
layout: post
title: Search Bar
tags: iOS
categories: Dev
---

## Search Bar

---

#### 1. [Customized SearchBar TextField](#p2)
#### 2. [Locking a UISearchBar on the top](#p2)
#### 3. [Bar button item that expands to a search bar](#p3)
#### 4. [Usage of UISearchController](#p4)

---

<br>

> #### 1. Customized SearchBar TextField {#p1}

Use key-value to get the textfield from `UISearchBar` subview hierarchy and set its properties as needed like background color, border color, etc.

```objc
UITextField *txfSearchField = [searchbar valueForKey:@"_searchField"];
[txfSearchField setBackgroundColor:[UIColor whiteColor]];
[txfSearchField setLeftView:UITextFieldViewModeNever];
[txfSearchField setBorderStyle:UITextBorderStyleRoundedRect];
txfSearchField.layer.borderWidth = 8.0f; 
txfSearchField.layer.cornerRadius = 10.0f;
txfSearchField.layer.borderColor = [UIColor clearColor].CGColor;
```

---

<br>

> #### 2. Locking a UISearchBar on the top {#p2}

Locking a UISearchBar to the top of a UITableView like Game Center.

```objc
-(void)scrollViewDidScroll:(UIScrollView *)scrollView 
{
    UISearchBar *searchBar = searchDisplayController.searchBar;
    CGRect rect = searchBar.frame;
    rect.origin.y = MIN(0, scrollView.contentOffset.y);
    searchBar.frame = rect;
}
```

---

<br>

> #### 3. Bar button item that expands to a search bar {#p3}

```swift
class ExpandableView: UIView {

    override init(frame: CGRect) {
        super.init(frame: frame)
        backgroundColor = .green
        translatesAutoresizingMaskIntoConstraints = false

    }

    required init?(coder aDecoder: NSCoder) {
        super.init(coder: aDecoder)
    }

    override var intrinsicContentSize: CGSize {
        return UILayoutFittingExpandedSize
    }
}


class ViewController: UIViewController {

    var leftConstraint: NSLayoutConstraint!

    override func viewDidLoad() {
        super.viewDidLoad()

        assert(navigationController != nil, "This view controller MUST be embedded in a navigation controller.")

        // Expandable area.
        let expandableView = ExpandableView()
        navigationItem.titleView = expandableView

        // Search button.
        navigationItem.rightBarButtonItem = UIBarButtonItem(barButtonSystemItem: .search, target: self, action: #selector(toggle))

        // Search bar.
        let searchBar = UISearchBar()
        searchBar.translatesAutoresizingMaskIntoConstraints = false
        expandableView.addSubview(searchBar)
        leftConstraint = searchBar.leftAnchor.constraint(equalTo: expandableView.leftAnchor)
        leftConstraint.isActive = false
        searchBar.rightAnchor.constraint(equalTo: expandableView.rightAnchor).isActive = true
        searchBar.topAnchor.constraint(equalTo: expandableView.topAnchor).isActive = true
        searchBar.bottomAnchor.constraint(equalTo: expandableView.bottomAnchor).isActive = true
    }

    @objc func toggle() {

        let isOpen = leftConstraint.isActive == true

        // Inactivating the left constraint closes the expandable header.
        leftConstraint.isActive = isOpen ? false : true

        // Animate change to visible.
        UIView.animate(withDuration: 1, animations: {
            self.navigationItem.titleView?.alpha = isOpen ? 0 : 1
            self.navigationItem.titleView?.layoutIfNeeded()
        })
    }
}
```

---

<br>

> #### 4. Usage of UISearchController {#p4}

**1).** 创建用于展示搜索结果的控制器和搜索栏

```objc
//  用于展示搜索结果的控制器
ResultDisplayController *result = [[ResultDisplayController alloc]init];
//  创建搜索框
UISearchController *search = [[UISearchController alloc]initWithSearchResultsController:result];
self.tableView.tableHeaderView = search.searchBar;
search.searchResultsUpdater = result;
self.searchController = search;
```

**2).** 在搜索结果的控制器中去实现 `**UISearchResultsUpdating**` 协议

```objc
@interface ResultDisplayController : UIViewController<UISearchResultsUpdating>

@end
```

```objc
@implementation ResultDisplayController

#pragma mark - UISearchResultsUpdating

- (void)updateSearchResultsForSearchController:(UISearchController *)searchController {
    //  这里处理过滤数据源的逻辑
    NSString *inputStr = searchController.searchBar.text ;
    if (self.results.count > 0) {
        [self.results removeAllObjects];
    }
    for (NSString *str in self.datas) {
        if ([str.lowercaseString rangeOfString:inputStr.lowercaseString].location != NSNotFound) {
            [self.results addObject:str];
        }
    }
    //  然后刷新表格
    [self.tableView reloadData];
}

@end
```

**Search Scope**

The iOS UISearchBar makes it dead easy to add a scope bar so users can change what they’re searching through. 
In Interface Builder we can add scope titles of search bar. Also we need to adjust our filter function to use the
 scope. We need to get the current scope from the search bar.

The search should also reload when the scope changes, not just when the search text changes so we’ll have to implement the searchDisplayController:shouldReloadTableForSearchScope: fuction:


```swift
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
```

**Setting Up searchController‘s Parameters**

**1).** searchResultsUpdater is a property on UISearchController that conforms to the new protocol, UISearchResultsUpdating. With this protocol, UISearchResultsUpdating will inform your class of any text changes within the UISearchBar.

```swift
searchController.searchResultsUpdater = self
```

**2).** By default, UISearchController obscures the view controller containing the information you’re searching. This is useful if you’re using another view controller for your searchResultsController. In this instance, you’ve set the current view to show the results, so you don’t want to obscure your view.

```swift
searchController.obscuresBackgroundDuringPresentation = false
```

**3).** Here, you set the placeholder to something that’s specific to this app.

```swift
searchController.searchBar.placeholder = "Search Candies"
```

**4).** New for iOS 11, you add the searchBar to the navigationItem. This is necessary because Interface Builder is not yet compatible with UISearchController.

```swift
navigationItem.searchController = searchController
```

**5).** Finally, by setting definesPresentationContext on your view controller to true, you ensure that the search bar doesn’t remain on the screen if the user navigates to another view controller while the UISearchController is active.

```swift
definesPresentationContext = true
``

---

<br>

#### Reference links

* [系统UISearchController详解](https://www.jianshu.com/p/aa9a153a5b58)
* [Adding a Search Bar to a Table View in Swift](https://grokswift.com/swift-tableview-search-bar/)
* [UISearchController Tutorial: Getting Started](https://www.raywenderlich.com/4363809-uisearchcontroller-tutorial-getting-started)
