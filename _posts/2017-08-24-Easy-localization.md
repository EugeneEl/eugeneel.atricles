---
layout: post
title: Easy localization in iOS
---

Today I'd like to cover such topic as app localization. Actually it is simple task however I intend sharing some useful approaches which make localization even more easier and pleasant.   

---

## 1. Localization is essential from start
It doesn't matter your app has only one language now. Mobile app development assumes rapid and arbitrary changes thus requirement for new language might appear in every moment. 

It would be much better to just give customer a list with translations 
than to stop all current tasks replacing hard coded texts all over the app.

That's why localization is must from start. 
If code has hard coded string instead of localization - pull/merge request must be rejected.
Even when you leave your project - people supporting it will mention you in good way.

## 2. Keep all localizations in code

It's personal attitude. I prefer to see all localizations in code because for me it is obvious.
And I focus on making my UI reusable so localizing storyboards is not suitable technique to my view.

## 3. Use extensions to wrap your localization
I recommend to have a look on the powerful library [Localize-Swift](https://github.com/marmelroy/Localize-Swift). It has a set of useful extensions for localization.

Instead of this 
```swift
textLabel.text  = NSLocalizedString("LoginScene.HelloTextLabel.Text", comment: "") 
```

you can simply use *localized()* func 

```swift
textLabel.text = "LoginScene.HelloTextLabel.Text".localized()
```

## 4. Localize your info.plist for permissions description
If you ask permissions for location or camera - you should  provide usage descriptions. 
In multi language app you should provide proper localizations. For this you need to create localized version of your info.plist. Here is proper topic on [stackoverflow](https://stackoverflow.com/questions/25736700/how-to-localise-a-string-inside-the-ios-info-plist-file)

Actually it will be separate file InfoPlist.strings.

![ScreenShot]( http://s019.radikal.ru/i618/1708/45/a5261922ec86.png)

And you will need to provide translations for different permission keys

```swift
"NSCameraUsageDescription" = "We need this to take pictures.";
```

Don't forget to add "Localized resources can be mixed" = YES in your info.plist to enable localizations.


## 5. Localize images instead of complicated layout

Sometimes it is much easier to prepare localized version of image than struggling with different layout for multiple languages.


## 6. Prefer intristic content size over hardcoded values

When you need to localize such UI elements as UIBarButtonItem instead of magic numbers or manual text size calculation rely on UIView intristic content size property. You can read more about it in my previous [post](https://eugenegoloboyar.github.io/2017/04/17/UI-tips-and-hacks)


## 7. Create shared Google table for localization
Working on different big project with localization I found out that copy-pasting localization changes is time consuming. Even more irritating is to check old/new version and fix such annoying bugs as broken characters encoding. 

So firstly you can create shared Google doc table with all translations. 
To do this in automatic way there is a pretty tool [Babelish](https://github.com/netbe/Babelish) which can convert iOS localizable strings to *.csv.

After installing Babelish tool *cd /your_localizable_string_file_location* and run this command:

```
strings2csv --filenames=Localizable.strings

Creating translations.csv
Done
```

Than you can import your translations.csv in google doc. 

Now we have all translations in one place. If customer wants to apply changes he can just mark proper cells in Google doc. 
It is better to have one sheet with all translations and separate sheets for each language for easy export.

![ScreenShot](http://s009.radikal.ru/i308/1708/51/546bd40532dd.png)


This makes changes on our side much easier.
In case we have a lot of changes or even new language - no problem. We can export all new translations to csv file for proper language and convert it to iOS localized strings with already mentioned Babelish tool.

Your google doc sheet should have the following struct for converting it to .strings, pay attention to the name of columns : *Variables*, *English*

![ScreenShot](http://s013.radikal.ru/i325/1708/a7/0b9a3c2ba1a6.png) 
 
Now we need to export this sheet as .csv and than convert it with command:

```
cd /your .csv file location

babelish csv2strings --filename=Translations.csv --langs English:en

Created 1 files.
List of created files:
//...Localizable.strings
```

It's worth to mention that Babelish also can be used for converting Android localization resources, json etc. 

Needless to say that this approach can significantly save your time. All translations are in one place. Changes can be applied easily without reviewing localization.txt 100500 times.
Translations for new languages can be converted to localized strings in one second.  

---
## That's all for now

The MIT License (MIT)

Copyright © 2017 EugeneiOS.

