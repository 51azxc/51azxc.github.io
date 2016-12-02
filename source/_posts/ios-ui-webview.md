title: "iOS WebView"
date: 2015-04-10 17:24:32
tags: "webview"
categories: ["iOS","Objective-C"]
---

### UIWebView

#### 基本使用

> [UIWebView网页视图—IOS开发](http://blog.csdn.net/iukey/article/details/7299763)
> [UIWebView的一些用法总结](http://my.oschina.net/hmj/blog/147507)

```objc
@interface ViewController : UIViewController<UIWebViewDelegate>

@interface ViewController ()
{
    UIWebView *myWebView;
}

@end

@implementation ViewController

- (void)viewDidLoad {
    [super viewDidLoad];

    CGRect bounds = [[UIScreen mainScreen]applicationFrame];
    myWebView = [[UIWebView alloc] initWithFrame:bounds];
    [self.view addSubview:myWebView];
    //网页自适配屏幕
    myWebView.scalesPageToFit = YES;
    //设置背景透明
    myWebView.backgroundColor = [UIColor clearColor];
    myWebView.opaque = NO;
    //加载网络资源
    NSURL *url = [NSURL URLWithString:@"http://www.baidu.com"];
    NSURLRequest *request = [NSURLRequest requestWithURL:url];
    [myWebView loadRequest:request];
    /*
    //加载本地资源
    NSString *path = [[NSBundle mainBundle] pathForResource:@"test" ofType:@"html"];
    NSURL *url = [NSURL fileURLWithPath:path];
    [myWebView loadRequest:[NSURLRequest requestWithURL:url]];
     */
    //去除划动到边缘的反弹效果
    myWebView.scrollView.bounces = NO;
    //调用javascript代码
    [myWebView stringByEvaluatingJavaScriptFromString:@"function test(){alert(1);}"];
    //指定自身为UIWebViewDelegate
    myWebView.delegate = self;
}
//发生页面跳转时调用的对象，返回YES则继续，NO则终止
-(BOOL)webView:(UIWebView *)webView shouldStartLoadWithRequest:(NSURLRequest *)request navigationType:(UIWebViewNavigationType)navigationType
{
    //这里可以通过request获取url来得到相关的内容，如果需要js调用native对象，一般的实现方式都是通过一个隐藏的iframe的页面加特殊前缀的跳转然后再到此方法内获取再进行下一步操作
    //navigationType有如下枚举
   // UIWebViewNavigationTypeLinkClicked，用户触击了一个链接。
   // UIWebViewNavigationTypeFormSubmitted，用户提交了一个表单。
   // UIWebViewNavigationTypeBackForward，用户触击前进或返回按钮。
   // UIWebViewNavigationTypeReload，用户触击重新加载的按钮。
   // UIWebViewNavigationTypeFormResubmitted，用户重复提交表单
   // UIWebViewNavigationTypeOther，发生其它行为。
    return YES;
}
//载入网页之前调用的方法
-(void)webViewDidStartLoad:(UIWebView *)webView
{
    NSLog(@"start");
}
//成功载入网页之后调用的方法
-(void)webViewDidFinishLoad:(UIWebView *)webView
{
    NSLog(@"end");
}
//载入网页失败时调用的方法
-(void)webView:(UIWebView *)webView didFailLoadWithError:(NSError *)error
{
    NSLog(@"%@",[error localizedDescription]);
}

```

---

#### 页面缩放

> [改变UIWebView的缩放系数](http://blog.csdn.net/kmyhy/article/details/7198920)

通过js实现，然后利用`stringByEvaluatingJavaScriptFromString`加载js

```js
var metaElement = document.createElement("meta");
metaElement.name = "viewport";
metaElement.content = "width=device-width,initial-scale=1.28,maximum-scale=1.28,minimum-scale=1.28";
var head = document.getElementsByTagName("head")[0];
head.appendChild(metaElement);
```

相关的参数有

* __minimum-scale__: 允许缩放的最小倍数。默认为0.25，允许值为0－10。
* __maximum-scale__: 运行缩放的最大倍数。默认1.6，允许值为0－10。
* __initial-scale__: 当web页被加载，还未被用户缩放之前默认的缩放系数。默认值是自动根据页面大小和可用区域计算出来的，但这个值最终会在最小倍数到最大倍数之间。
* __user-scalable__: 是否运行用户缩放该web页。
* __width__: viewpoint的宽。
* __height__: viewport的高。通常是根据width计算的。

----

#### 解析HTTPS请求

> [关于使用UIWebView加载HTTPS站点出现NSURLErrorDomain code=-1202](http://blog.csdn.net/pingchangtan367/article/details/8264858)


当UIWebview加载https的站点时webview报NSURLErrorDomain code=-1202错误时,https使用超文本安全传输协议，即超文本传输协议（HTTP）和SSL/TLS的组合，用以提供加密通讯及对网络服务器身份的鉴定。当我们的服务器使用自我签名证书时，而UIWebView不允许使用自签名证书，所以导致加载失败。我们可以使用NSURLConnection通过它的代理canAuthenticateAgainstProtectionSpace可以允许这种情况,从而通过它进行认证。

```objc
@interface ViewController : UIViewController<UIWebViewDelegate,NSURLConnectionDelegate>

@interface ViewController ()
{
    UIWebView *myWebView;
    BOOL authenticated;
    NSURLConnection *urlConn;
    NSURLRequest *urlRequest;
}

@end

@implementation ViewController

- (void)viewDidLoad {
    [super viewDidLoad];

    CGRect bounds = [[UIScreen mainScreen]applicationFrame];
    myWebView = [[UIWebView alloc] initWithFrame:bounds];
    [self.view addSubview:myWebView];
    myWebView.scalesPageToFit = YES;

    NSURL *url = [NSURL URLWithString:@"http://www.baidu.com"];
    urlRequest = [NSURLRequest requestWithURL:url];
    [myWebView loadRequest:urlRequest];
    myWebView.delegate = self;
}

-(BOOL)webView:(UIWebView *)webView shouldStartLoadWithRequest:(NSURLRequest *)request navigationType:(UIWebViewNavigationType)navigationType
{
    if (!authenticated) {
        authenticated = NO;
        urlConn = [[NSURLConnection alloc] initWithRequest:urlRequest delegate:self];
        [urlConn start];
        
        return NO;
    }
    return YES;
}
-(void)connection:(NSURLConnection *)connection didReceiveAuthenticationChallenge:(NSURLAuthenticationChallenge *)challenge
{
    NSLog(@"WebController Got auth challange via NSURLConnection");
    if ([challenge previousFailureCount] == 0) {
        authenticated = YES;
        NSURLCredential *credential = [NSURLCredential credentialForTrust:challenge.protectionSpace.serverTrust];
        [challenge.sender useCredential:credential forAuthenticationChallenge:challenge];
    } else {
        [[challenge sender] cancelAuthenticationChallenge:challenge];
    }
}

- (void)connection:(NSURLConnection *)connection didReceiveResponse:(NSURLResponse *)response;
{
    
    NSLog(@"WebController received response via NSURLConnection");
    authenticated =YES;
    [myWebView loadRequest: urlRequest];
    // Cancel the URL connection otherwise we double up (webview + url connection, same url = no good!)
    [urlConn cancel];
    
}

-(BOOL)connection:(NSURLConnection *)connection canAuthenticateAgainstProtectionSpace:(NSURLProtectionSpace *)protectionSpace
{
    return [protectionSpace.authenticationMethod isEqualToString: NSURLAuthenticationMethodServerTrust];
}

@end
```

----

#### UIWebView cookie

> [iOS UIWebView 通过 cookie 完成自动登录验证](http://blog.csdn.net/assholeu/article/details/38585243)
> [在UIWebView中设置cookie](http://blog.csdn.net/wangyx810328/article/details/8752295)

```objc
- (void)setCookie
{
    NSMutableDictionary *cookieDict = [NSMutableDictionary dictionary];
    
    [cookieDict setObject:@"Set-Cookie" forKey:NSHTTPCookieName];
    [cookieDict setObject:@"Test" forKey:NSHTTPCookieValue];
    [cookieDict setObject:@"www.baidu.com" forKey:NSHTTPCookieDomain];
    [cookieDict setObject:@"/content" forKey:NSHTTPCookiePath];
    [cookieDict setObject:@"1.0" forKey:NSHTTPCookieVersion];
    [cookieDict setObject:[[NSDate date] dateByAddingTimeInterval:123456789] forKey:NSHTTPCookieExpires];
    
    NSHTTPCookie *myCookie = [NSHTTPCookie cookieWithProperties:cookieDict];
    [[NSHTTPCookieStorage sharedHTTPCookieStorage] setCookie:myCookie];
    
}

- (void)deleteCookie
{
    NSHTTPCookieStorage *myCookie = [NSHTTPCookieStorage sharedHTTPCookieStorage];
    NSArray *cookieArray = [myCookie cookiesForURL: [NSURL URLWithString:propValue]];
    for (NSHTTPCookie *cookie in cookieArray) {
        [myCookie deleteCookie:cookie];
    }
}

```

----

### 用WKWebView替换UIWebView

> [WKWebView 实例](https://cnbin.github.io/blog/2015/09/14/wkwebview-shi-li/)
> [使用WKWebView替换UIWebView](http://www.jianshu.com/p/6ba2507445e4)
> [UIWebView和WKWebView的使用及js交互](http://liuyanwei.jumppo.com/2015/10/17/ios-webView.html)
> [Allow unverified ssl certificates in WKWebView](http://stackoverflow.com/questions/27100540/allow-unverified-ssl-certificates-in-wkwebview)

**iOS8**之后，苹果推出了`WKWebView`来代替`UIWebView`组件。如要使用`WKWebView`,需要引入`#import <WeKit/WebKit.h>`.

```objc
@interface LoginWebViewController : UIViewController<WKNavigationDelegate, WKUIDelegate>

@property(nonatomic, strong) WKWebView *myWebView;

@end

@implementation LoginWebViewController

@synthesize myWebView;

- (void)viewDidLoad {
    [super viewDidLoad];
    
    CGRect rect = [[UIScreen mainScreen] applicationFrame];
    myWebView = [[WKWebView alloc] initWithFrame:rect];
    
    NSURLRequest *urlRequest = [NSURLRequest requestWithURL:[NSURL URLWithString:@"https://www.google.com"]];
    
    NSLog(@"url: %@", urlRequest.URL.absoluteString);
    
    //加载网页，与UIWebView一致
    [myWebView loadRequest:urlRequest];
    
    myWebView.UIDelegate = self;
    myWebView.navigationDelegate = self;
    
    [self.view addSubview:myWebView];
}

//页面开始加载时调用
- (void)webView:(WKWebView *)webView didStartProvisionalNavigation:(WKNavigation *)navigation {

}
//内容开始返回时调用
- (void)webView:(WKWebView *)webView didCommitNavigation:(WKNavigation *)navigation {

}
//页面加载完成之后调用
- (void) webView:(WKWebView *)webView didFinishNavigation:(WKNavigation *)navigation {
    
}
//页面加载失败时调用
- (void) webView:(WKWebView *)webView didFailProvisionalNavigation:(WKNavigation *)navigation withError:(NSError *)error {
    NSLog(@"didFailProvisionalNavigation withError: %ld, %@", (long)[error code], [error localizedDescription]);
}

- (void)webView:(WKWebView *)webView didFailNavigation:(WKNavigation *)navigation withError:(NSError *)error {
    NSLog(@"didFailNavigation withError: %ld, %@", (long)[error code], [error localizedDescription]);
}

- (void)webView:(WKWebView *)webView didReceiveAuthenticationChallenge:(NSURLAuthenticationChallenge *)challenge completionHandler:(void (^)(NSURLSessionAuthChallengeDisposition, NSURLCredential * _Nullable))completionHandler {
    //将没有验证的证书设为可信    
    NSURLCredential *credential = [[NSURLCredential alloc] initWithTrust:challenge.protectionSpace.serverTrust];
    completionHandler(NSURLSessionAuthChallengeUseCredential, credential);
}
//在发送请求之前，决定是否跳转
- (void)webView:(WKWebView *)webView decidePolicyForNavigationAction:(WKNavigationAction *)navigationAction decisionHandler:(void (^)(WKNavigationActionPolicy))decisionHandler {

    NSURL *url = navigationAction.request.URL;
    if ([[url host] isEqualToString:@"localhost"]) {
        //不允许跳转
        decisionHandler(WKNavigationActionPolicyCancel);
    } else {
        //允许跳转
        decisionHandler(WKNavigationActionPolicyAllow);
    }
}
//在收到响应后，决定是否跳转
- (void)webView:(WKWebView *)webView decidePolicyForNavigationResponse:(WKNavigationResponse *)navigationResponse decisionHandler:(void (^)(WKNavigationResponsePolicy))decisionHandler {
    if ([navigationResponse.response isKindOfClass:[NSHTTPURLResponse class]]) {
        NSHTTPURLResponse *response = (NSHTTPURLResponse *)navigationResponse.response;
        NSURL *url = navigationResponse.response.URL;
        if (response.statusCode == 302 && [[url host] isEqualToString:@"localhost"]) {
            //不允许跳转
            decisionHandler(WKNavigationResponsePolicyCancel);
        }
    }
    //允许跳转
    decisionHandler(WKNavigationResponsePolicyAllow);
}
//接收到服务器跳转请求之后调用
- (void)webView:(WKWebView *)webView didReceiveServerRedirectForProvisionalNavigation:(WKNavigation *)navigation {
}

@end
```