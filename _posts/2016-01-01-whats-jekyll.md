---
layout: post
title: 7 Useful tips and hacks for iOS UI.
---

Building UI in mobile applications is one of the most time consuming tasks. Soemtimes we struggling with small things
to make our customers and designers happy. Let's have a look on a set of small tips to make your work on UI less painfull.    

---

1. Customize your navigation bar UI right.

When navigation bar has different design on your screens it is quite common to see the following code in each UIViewController instance:

```
    override func viewWillAppear(_ animated: Bool) {
    super.viewWillAppear(animated)

        UINavigationBar.appearance().tintColor = .blue
        UINavigationBar.appearance().barTintColor = .red
    }
```

You should remember that appearance acts as proxy for a class. So your settings will be applied to all navigation bars - even an instance of MFMailComposeViewController. So it might have incorrect navigation bar color.
And needless to say that this code will be duplicated in a lot of your view controllers. 


Let's create some struct which will describe your navigation bar UI.

```
struct NavigationBarUI {
    let tintColor: UIColor
    let barTintColor: UIColor
}
```

Now it has only tintColor and barTintColor, but you can extend it for your personal use, adding smth like 
title, boolean flag if isSeparatorVisible etc.

And let's create some BaseViewController class which will have var for of NavigationBarUI type so you can override it in your controllers. 
Actually inheritance is the simplest, but not the best approach. When you use inheritance you establish the strongest 
relationship between objects. So in order to apply it for some third party view controllers you will need to subclass them 
from our BaseViewController.     

```

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

Now let's create an instance of TestViewController which will be a subclass of our BaseViewController.

```
class TestViewController: BaseViewController {

    override var navigationBarUI: NavigationBarUI? {
        return NavigationBarUI(tintColor: UIColor.blue,
                               barTintColor: UIColor.purple)
    }
}
```

Now we don't need to duplicate UI setup in each UIViewController.

There is another question we need to cover when deailing with navigation bar. 
Question "why my status bar style doesn't work" is supposed to be one of the top most when we speak about navigation bar design.
It is covered greatly in Andy Matuschak's article [Let's Play: Refactor the Mega Controller!](https://realm.io/news/andy-matuschak-refactor-mega-controller/)

The main thing is that navigation controller doesn't normally ask its children for a preferred status bar style. It manages its own.
So we need to provide status bar style for navigation controller which holds 

Let's create BaseNavigationController class where we can update UINavigationController status bar style.

```
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

Now we can add to our BaseViewController another property for statusBarStyle. 

```
var statusBarStyle: UIStatusBarStyle {
        return .lightContent
}
```

We will try to pass it to the instance of out base navigationController in our confifureUI() function. 

```
if let nc = navigationController as? BaseNavigationViewController {
   nc.statusBarStyle = statusBarStyle
}
```

And take into an account case when controller isn't embedded in navigationController.


```
// MARK: - StatusBarStyle
    
override var preferredStatusBarStyle: UIStatusBarStyle {
   return statusBarStyle
}
```

As a small tip let's extend our NavigationBarUI structure for flag indicating should navigationBar have black separator.

struct NavigationBarUI {
    let tintColor: UIColor
    let barTintColor: UIColor
    let isSeparatorVisible: Bool
}

One of popular approaches is looping through all subviews of navigation bar and removing UIImageView.
Something like this:

```
for parent in self.navigationController!.navigationBar.subviews {
        for childView in parent.subviews {
            if(childView is UIImageView) {
                childView.removeFromSuperview()
            }
        }
}
```

This approach is error proned because it relies on private view hierarchy. If it changes it won't work anymore.

While the correct way is just to set empty image to the shadow image of navigation bar.

```
if !ui.isSeparatorVisible {
   let img = UIImage()
   navigationController?.navigationBar.shadowImage = img
   navigationController?.navigationBar.setBackgroundImage(img, for: UIBarMetrics.default)
} 
```

---

2. Use intristic content size instead of hardcoded values.

When we create custom UIBarItemButtons and want them to fit texts in different languages it is quite common to see smth like this:

```
fileprivate lazy var rightBarItem: UIBarButtonItem = {
   let rightButton = UIButton()
   rightButton.addTarget(self, action: #selector(TestViewController.save), for: .touchUpInside)
   rightButton.setTitle("Some very big textttttttttttttttt", for: .normal)
   rightButton.frame.size.width = 100500

   return UIBarButtonItem(customView: rightButton)
 }()
 ```

Another naive approach is to calcluate text size manually.

Let's have a look at UIView property intrinsicContentSize. As documentation noted "Custom views typically have content that they display of which the layout system is unaware." 
In this case we can rely on UIButton which calculates size of its content (in our case text) so after text is set we can know 
how big is it. 


> Jekyll is a simple, blog aware, static site generator. It takes a template directory [...] and spits out a complete, static website suitable for serving with Apache or your favorite web server. This is also the engine behind GitHub Pages, which you can use to host your project’s page or blog right here from GitHub.

It's an immensely useful tool. Find out more by [visiting the project on GitHub](https://github.com/jekyll/jekyll).
