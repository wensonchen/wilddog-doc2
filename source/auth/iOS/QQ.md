title: QQ 登录
---

通过集成 QQ 登录，你可以让你的用户使用他们的 QQ 帐号来进行 Wilddog 身份认证。

登录的用户可以访问野狗实时数据库中用户登录受限的数据。？？这句需要改

### 开始前的准备工作
1. 创建野狗应用。将下面 pod 添加到 `Podfile` 文件中：
```
pod 'Wilddog/Auth'
```
2. 在 [QQ 开放平台管理中心](http://op.open.qq.com/ios_appinfov2/detail?appid=111)，获取应用的 **App ID** 和 **App Secret**。
3. 在野狗控制面板中打开 QQ 登录方式:
  * 在野狗控制面板中选择 ”身份认证“->登录方式。
  * 点击 QQ 登录开关，点击配置，输入 QQ 帐号 **APP ID** 和 **App Secret**。

### Wilddog 身份认证
1. 参考 [QQ API 调用说明](http://wiki.open.qq.com/wiki/IOS_API%E8%B0%83%E7%94%A8%E8%AF%B4%E6%98%8E) 将 QQ 登录集成到认证的应用中。当初始化 `TencentOAuth` 对象时，设置 delegate 来接收登录和登出事件:
```
_qqOAuth = [[TencentOAuth alloc] initWithAppId:@"your QQ App ID"
                                   andDelegate:self];
NSArray *_permissions =  [NSArray arrayWithObjects:@"get_user_info", @"get_simple_userinfo", @"add_t", nil];
[qqOAuth authorize:_permissions inSafari:NO];
```
在你的 delegate 中，实现 `tencentDidLogin` 方法。
```
 - (void)tencentDidLogin {
    // ...
 }
```
2. 导入 WilddogAuth 模块：
```
@import WilddogAuth;
```
3. 初始化 `WDGAuth` 对象：
```
WDGAuth *auth = [WDGAuth authWithAppID:@"your-wilddog-appid"];
```
4. QQ 登录成功后，在 `tencentDidLogin` 方法中通过 _qqOAuth 对象获取 QQ access token 来生成 Wilddog 凭据：
```
WDGAuthCredential *credential = 
[WDGQQAuthProvider credentialWithAccessToken:_qqOAuth.accessToken];
```
5. 最后，使用 Wilddog 凭据来进行 Wilddog 用户认证：
```
[auth signInWithCredential:credential
                completion:^(WDGUser *user, NSError *error) {
                            // ...
                          }];
```

### 后续步骤

无论你采用哪种登录方式，用户第一次登录后，野狗服务器都会生成一个唯一的 Wilddog ID 来标识这个帐户，使用这个 Wilddog ID，可以在你 APP 中确认每个用户的身份。配合 [规则表达式](https://z.wilddog.com/rule/quickstart)，`auth` 还可以控制野狗实时数据库的用户访问权限。

* 在你的应用中，你可以通过 WDGUser 来获取用户的基本属性。参考 [管理用户](https://aoqishen.gitbooks.io/auth/content/iOS/guides/%E7%AE%A1%E7%90%86%E7%94%A8%E6%88%B7.html)。
* 在你的野狗实时数据库 [规则表达式](https://z.wilddog.com/rule/quickstart) 中，你可以获取到这个登录后生成的唯一用户 Wilddog ID， 通过他可以实现控制用户对数据的访问权限。

你还可以通过 [链接多种登录方式](https://aoqishen.gitbooks.io/auth/content/iOS/guides/%E9%93%BE%E6%8E%A5%E5%A4%9A%E7%A7%8D%E7%99%BB%E5%BD%95%E6%96%B9%E5%BC%8F.html) 来实现不同的登录方式登录同一个帐号。

#####调用 [signOut:]() 退出登录：
```
NSError *error;
[[WDGAuth authWithAppID:@"your-appid"] signOut:&error];
if (!error) {
  // 退出登录成功
}
```
可能发生的错误，请参考 [处理错误](https://aoqishen.gitbooks.io/auth/content/iOS/guides/%E5%A4%84%E7%90%86%E9%94%99%E8%AF%AF.html)。