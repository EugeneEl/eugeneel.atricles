---
layout: post
title: 7 Useful tips and hacks for iOS UI
---

Building UI in mobile applications is one of the most time consuming part. Sometimes we struggle for hours with small things to make our customers and designers happy. Let's have a look on a set of tips to make work on UI less painfull.    

---

## 1. Customize your navigation bar UI right

When navigation bar has different design on your screens we regurarly see the following code in each `UIViewController` subclass:

```swift
    override func viewWillAppear(_ animated: Bool) {
        super.viewWillAppear(animated)

        UINavigationBar.appearance().tintColor = .blue
        UINavigationBar.appearance().barTintColor = .red
    }
```

You should remember that [appearance](https://developer.apple.com/reference/uikit/uiappearance) acts as proxy for a class. So your settings will be applied to all navigation bars, including MFMailComposeViewController which can led to irritating errors.
Needless to say that this code will be duplicated in a lot of your viewcontrollers. 

Let's declare some struct which will describe our navigation bar UI.

```swift
struct NavigationBarUI {
    let tintColor: UIColor
    let barTintColor: UIColor
}
```

Now it has only tintColor and barTintColor, but you can extend it for your personal use, adding something like localized navigation title, boolean flag isSeparatorVisible etc.

And let's create some `BaseViewController` class which will have a var of `NavigationBarUI` type so you can override it in your controllers. 

Actually inheritance is the simplest, but not the best approach. Using inheritance you establish the strongest relationship between objects. You will need to subclass your third party viewcontroller from `BaseViewController`.     

```swift
class BaseViewController: UIViewController {
    
    var navigationBarUI: NavigationBarUI? {
        return nil
    }
    
    // MARK: - Lifecycle
    
    override func viewWillAppear(_ animated: Bool) {
        super.viewWillAppear(animated)
        
        configureUI()
    }
    
    // MARK: - Private
    
    fileprivate func configureUI() {
        if let ui = navigationBarUI {
            navigationController?.navigationBar.tintColor = ui.tintColor
            navigationController?.navigationBar.barTintColor = ui.barTintColor
        }
    }
}
```

Now let's create `TestViewController` which will be a subclass of our `BaseViewController`.

```swift
class TestViewController: BaseViewController {

    override var navigationBarUI: NavigationBarUI? {
        return NavigationBarUI(tintColor: UIColor.blue,
                               barTintColor: UIColor.purple)
    }
}
```

We don't need to duplicate UI setup in each UIViewController.

There is another question worth attention while working with navigation bar. 
Question _why my status bar style doesn't work_ supposed to be the top most one.
It is covered greatly in Andy Matuschak's article [Let's Play: Refactor the Mega Controller!](https://realm.io/news/andy-matuschak-refactor-mega-controller/)

The main thing is that navigation controller doesn't normally ask its children for a preferred status bar style. It manages its own one.
So we need to provide status bar style for navigation controller. 

Let's create `BaseNavigationController` class where we can update UINavigationController status bar style.

```swift
class BaseNavigationViewController: UINavigationController {
    
    var statusBarStyle: UIStatusBarStyle = .default {
        didSet {
            setNeedsStatusBarAppearanceUpdate()
        }
    }
    
    // MARK: - StatusBarStyle
    
    override var preferredStatusBarStyle: UIStatusBarStyle {
        return statusBarStyle
    }
}
```

Now we can add to our `BaseViewController` another property for statusBarStyle. 

```swift
var statusBarStyle: UIStatusBarStyle {
        return .lightContent
}
```

We will try to pass it to the instance of our base navigationController in confifureUI() function. 

```swift
if let nc = navigationController as? BaseNavigationViewController {
   nc.statusBarStyle = statusBarStyle
}
```

And take into an account case when controller isn't embedded in `UINavigationController`.


```swift
// MARK: - StatusBarStyle
    
override var preferredStatusBarStyle: UIStatusBarStyle {
   return statusBarStyle
}
```

As a small tip let's extend our `NavigationBarUI` structure for flag indicating should navigation bar have black separator.

```swift
struct NavigationBarUI {
    let tintColor: UIColor
    let barTintColor: UIColor
    let isSeparatorVisible: Bool
}
```

One of popular hacky approaches is looping through all subviews of navigation bar and removing the first found `UIImageView`.
Something like this:

```swift
for parent in self.navigationController!.navigationBar.subviews {
        for childView in parent.subviews {
            if(childView is UIImageView) {
                childView.removeFromSuperview()
            }
        }
}
```

This approach is error proned because it relies on private view hierarchy. If hierarchy of views changes it won't work anymore.

While the correct way is just to set empty image to the shadow image of navigation bar.

```swift
if !ui.isSeparatorVisible {
   let img = UIImage()
   navigationController?.navigationBar.shadowImage = img
   navigationController?.navigationBar.setBackgroundImage(img, for: UIBarMetrics.default)
} 
```

---

## 2. Use intristicContentSize over hardcoded values

When we create custom UIBarButtonItems and want them to fit texts in different languages it is quite common to see smth like this:

```swift
fileprivate lazy var rightBarItem: UIBarButtonItem = {
   let rightButton = UIButton()
   rightButton.addTarget(self, action: #selector(TestViewController.save), for: .touchUpInside)
   rightButton.setTitle("Some very big textttttttttttttttt", for: .normal)
   rightButton.frame.size.width = 100500

   return UIBarButtonItem(customView: rightButton)
 }()
 ```

Another naive approach is to calcluate text size manually.

Let's have a look at `UIView` property [intrinsicContentSize](https://developer.apple.com/reference/uikit/uiview/1622600-intrinsiccontentsize). According to documentation _custom views typically have content that they display of which the layout system is unaware._ 
In this case we can rely on `UIButton` which calculates size of its content (in our case text) so after text is set we can know how big is it. 
If we dealing with `UIImageView` it calcalutes intristicContentSize for its image.
In your custom view you can provide your own intristicContentSize.

Let's create custom button which will update its frame based on intristicContentSize.

```swift
class NavigationBarButton: UIButton {
    
    // MARK: - Override
    
    override func  setAttributedTitle(_ title: NSAttributedString?, for state: UIControlState) {
        super.setAttributedTitle(title, for: state)
        
        updadeSize()
    }
    
    override func setTitle(_ title: String?, for state: UIControlState) {
        super.setTitle(title, for: state)
        
        updadeSize()
    }
    
    // MARK: - Private
    
    fileprivate func updadeSize() {
        frame.size = intrinsicContentSize
    }
}
```

Now we can use our custom button which will resize with its content. 
No more magic numbers or manual calculations.    

```swift
fileprivate lazy var rightBarItem: UIBarButtonItem = {
   let rightButton = NavigationBarButton()
   rightButton.addTarget(self, action: #selector(TestViewController.save), for: .touchUpInside)
   rightButton.setTitle("Some very big textttttttttttttttt", for: .normal)

   return UIBarButtonItem(customView: rightButton)
 }()
 ```
---

## 3. Override intristicContentSize in your custom views

When building complicated UI with a lot of ebmedded or aggregated elements we usually need to provide 
height for each UITableViewCell. Luckily from iOS 8 we can use [self-sizing cells](https://developer.apple.com/library/content/documentation/UserExperience/Conceptual/AutolayoutPG/WorkingwithSelf-SizingTableViewCells.html) so iOS will automatically calclate entire cell size based on its content and constraints.
However in some cases we still need to provide cell height explicitly.
For example when cell also contains UITableView (aggregated cells).
Old-school way is to calculate height of all cells based on the tableView dataSource.

We can do it simpler:   

```swift
class IntristicTableView: UITableView {
    
    // MARK: - Override
    
    override var intrinsicContentSize : CGSize {
        layoutIfNeeded()
        return CGSize(width: UIViewNoIntrinsicMetric, height: contentSize.height);
    }
}
```

Now we can simply replace UITableViewClass in cell's xib with our custom one. And we don't need to calculate tableView height anymore. We just call [layoutIfNeeded](https://developer.apple.com/reference/uikit/uiview/1622507-layoutifneeded) to force height calclulation. And contentSize of UITableView will be enough to know embedded height.

---

## 4. Generic instantiation of custom views

Its is much better to instantiate custom views using their class names instead of hardcoded strings.
Quite easy to mistype wrong letter and get exception. 
Usually best practice is to keep name of your view's xib file the same as your custom view class.

And we can write smth like this:

```swift
class func loadView(_ nibName: String) -> UIView? {
      var loadedView: UIView?
      if let views = Bundle.main.loadNibNamed(nibName, owner: nil, options: nil) {
            loadedView = views.last as? UIView
      }
        
      return loadedView
}
```

```swift
loadedView = views.last as? UIView
```

or 

```swift
loadedView = views.first as? UIView
```

is quite error-proned approach.

Let's see what documentation tells us about [loadNibNamed:owner:options:](https://developer.apple.com/reference/foundation/nsbundle/1618147-loadnibnamed). It returns _an array containing the top-level objects in the nib file._ In xib file we can have more than one object. For exapmle an instance of NSObject with some dataSource logic.
So the correct what is to filter this array and search for our custom view:

```swift
protocol ViewInitializing {}

extension UIView: ViewInitializing {}

extension ViewInitializing where Self: UIView {
    static func instantiateView() -> Self {

        let loadedElements = Bundle.main.loadNibNamed(Self.viewName(), owner: nil, options: nil)!
        let loadedView = loadedElements.filter({$0 is Self}).first
        
        assert(loadedView != nil, "wrong nib name for view: \(Self.viewName)")

        return loadedView as! Self
    }
}

extension UIView {
    
    static func viewName() -> String {
        return String(describing: self)
    }
}
```

---

## 5. Generic instantiation of cells
We already covered generic instantiation of views. We can do the same for UITableViewCells.
Wouldn't it be great to have to instantiate your cells in generic way instead of using hardcoded strings for cell identifiers?

```swift
extension UITableViewCell {    
    static func cellIdentifier() -> String {
        return String(describing: self)
    }
}

protocol CellInitializing {
    
}

extension UITableViewCell: CellInitializing {}

extension CellInitializing where Self: UITableViewCell {
    static func dequeueFromTableView(_ tableView: UITableView) -> Self {
        let identifier = self.cellIdentifier()
        
        let cell = tableView.dequeueReusableCell(withIdentifier: identifier) as! Self
        return cell
    }
}
```

---
## 6. Use SnapKit for builing your layout in code

Sometimes when we are working on custom views we need to setup our constraints programmatically.
Instead of this 

```swift 
let myView = UIView()
myView.backgroundColor = UIColor.redColor()
self.view.addSubview(myView)
myView.translatesAutoresizingMaskIntoConstraints = false

view.addConstraint(NSLayoutConstraint(item: myView, attribute: .Top, 
                  relatedBy: .Equal, toItem: self.topLayoutGuide, 
                  attribute: .Bottom, multiplier: 1, constant: 0))
view.addConstraint(NSLayoutConstraint(item: myView, attribute: .Bottom, 
                  relatedBy: .Equal, toItem: self.bottomLayoutGuide, 
                  attribute:.Top, multiplier: 1, constant: 20))
view.addConstraint(NSLayoutConstraint(item: myView, attribute: .Width, 
                  relatedBy: .Equal, toItem: nil, 
                  attribute: .NotAnAttribute,multiplier: 1, constant: 300))
view.addConstraint(NSLayoutConstraint(item: myView, attribute: .TrailingMargin, 
                  relatedBy: .Equal, toItem: view, 
                  attribute: .TrailingMargin, multiplier: 1, constant: 0))
```

it can be simplier with [SnapKit](https://github.com/SnapKit/SnapKit)

```swift
view.snp.makeConstraints {[unowned self]  make in
  make.top.equalTo(self.anotherView.snp.top).inset(self.topOffset)
  make.centerX.equalTo(self.anotherView.snp.centerX)
  make.width.equalTo(self.viewWidth)
  make.height.equalTo(self.viewHeight)
}
```

---
## 7. Use StackView for building forms.

Forms supposed to be one of the most annoying party of UI. There are dozens of third party libraries 
for builing forms but none of them became a silver-bullet in practice. 

Before `UIStackView` we had only one way to build forms - using `UITableView`.
The most difficult part is to handle all cases in UITableViewDataSource methods. And if you missed some case you would defenitely result in wrong behaviour.
Things become even more complicated when we also have something like expand/collapse logic.

Luckily `UIStackView` provides amazing ability to create forms. You just need place and arrange 
your elements. For expand/collapse behaviour we can just hide/unhide particular subviews.
`UIStackView` does everything else for us. 
It is much easier than to calculate indexes for all cases in dataSource. 

However each tool should be used in right way. The same is true for `UIStackView`.
If you have a lot of content and it has similiar elements stackview might not be right choice.
It doesn't provide reuse mechanism as `UITableView`. If you create 20 views in code your scrolling might have poor user experience. 

---
## That's all for now

You can also view simple [demo](https://github.com/EugeneGoloboyar/SimpleUIProject)

The MIT License (MIT)

Copyright © 2017 EugeneiOS.

