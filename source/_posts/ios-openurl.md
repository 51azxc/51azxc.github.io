title: 在iOS应用中打开另一个应用
date: 2015-10-25 18:45:37
tags: ["openUrl"]
categories: ["iOS","swift"]
---

### 在应用中打开另一个应用

> [利用openURL，在IOS应用中打开另外一个应用](http://blog.sina.com.cn/s/blog_a170e5c80101gsdj.html)
> [canOpenUrl - This app is not allowed to query for scheme instragram iOS9](http://stackoverflow.com/questions/32870393/canopenurl-this-app-is-not-allowed-to-query-for-scheme-instragram-ios9)
> [iOS: Access app-info.plist variables in code](http://stackoverflow.com/questions/9530075/ios-access-app-info-plist-variables-in-code)

首先需要在`info.plist`注册自定义的`URL scheme`
```xml
<key>CFBundleURLTypes</key>
<array>
    <dict>
        <key>CFBundleURLSchemes</key>
        <array>
            <string>test1</string>
        </array>
    </dict>
</array>
```
在`iOS9`以后，还需要添加以下属性:
```xml
<key>LSApplicationQueriesSchemes</key>
<array>
    <string>test2</string>
</array>
```
这里意为应用可以打开`scheme`为`test2`的应用。
然后在代码中可以使用`openUrl`函数打开应用：
```objc
//获取scheme
let prefix = (Bundle.main.object(forInfoDictionaryKey: "LSApplicationQueriesSchemes") as! NSArray)[0]
let url = URL(string: "\(prefix):client_id=test1&scoe=wopi&platform=iOS&app=test1&action=12345")
if UIApplication.shared.canOpenURL(url! as URL) {   //判断能否打开
    //打开应用
    UIApplication.shared.openURL(url! as URL)
} else {
    print("can not go to this app!")
}
```
接下来应用可以在`AppDelegate`类中的`handleOpenUrl`方法中获取到传过来的参数:
```objc
func application(_ application: UIApplication, handleOpen url: URL) -> Bool {
    print(url)
    
    let viewController = self.window?.rootViewController
    let message = url.absoluteString
    let alertController = UIAlertController.init(title: "url", message: message, preferredStyle: .alert)
    alertController.addAction(UIAlertAction.init(title: "OK", style: .default, handler: nil))
    viewController?.present(alertController, animated: true, completion: nil)
    
    return true;
}
```

----

### 在网页中打开应用

在网页`<script>`块中直接定义跳转的`scheme`为应用自定义的`scheme`即可

```js
//如果无法打开应用则跳转到AppStore中
setTimeout(function () {
    window.location = "https://itunes.apple.com/cn/app/microsoft-word/id586447913"
}, 25);
window.location = "test1://";
```
