


登录服务引入头文件
#import <AuthenticationServices/AuthenticationServices.h>


2. iOS13代理声明

API_AVAILABLE(ios(13.0))
@interface LoginController () <ASAuthorizationControllerDelegate,ASAuthorizationControllerPresentationContextProviding>

@property (nonatomic, strong) ASAuthorizationAppleIDButton *appIdButton;

@end


3.显示Apple登录图标 添加到 view上
[self.view addSubview:self.appIdButton];



4.设置Apple图标的 像素 
self.appIdButton.frame = CGRectMake(100, 100, 100, 40);


5.  初始化苹果登录按钮
#pragma mark - get
//初始化苹果登录按钮
- (ASAuthorizationAppleIDButton *)appIdButton  API_AVAILABLE(ios(13.0)){
    if (!_appIdButton) {
        //ASAuthorizationAppleIDButtonStyleWhite  这个是底色
        _appIdButton = [ASAuthorizationAppleIDButton buttonWithType:ASAuthorizationAppleIDButtonTypeDefault style:ASAuthorizationAppleIDButtonStyleBlack];
        [_appIdButton addTarget:self action:@selector(didAppleIDBtnClicked) forControlEvents:UIControlEventTouchUpInside];
    }
    return _appIdButton;
}


6.用户点击Apple登录调取页面，并且添加到Windows上
#pragma mark ASAuthorizationControllerPresentationContextProviding
- (ASPresentationAnchor)presentationAnchorForAuthorizationController:(ASAuthorizationController *)controller  API_AVAILABLE(ios(13.0)){
    
    NSLog(@"调用展示window方法：%s", __FUNCTION__);
    // 返回window
    return self.view.window;
}


7.苹果登录 核心代码

- (void)didAppleIDBtnClicked {
    if (@available(iOS 13.0, *)) {
        //基于用户的Apple ID授权用户，生成用户授权请求的一种机制
        ASAuthorizationAppleIDProvider *appleIDProvider = [ASAuthorizationAppleIDProvider new];
        //授权请求AppleID
        ASAuthorizationAppleIDRequest *request = appleIDProvider.createRequest;
        [request setRequestedScopes:@[ASAuthorizationScopeFullName, ASAuthorizationScopeEmail]];
        //由ASAuthorizationAppleIDProvider创建的授权请求 管理授权请求的控制器
        ASAuthorizationController *controller = [[ASAuthorizationController alloc] initWithAuthorizationRequests:@[request]];
        //设置授权控制器通知授权请求的成功与失败的代理
        controller.delegate = self;
        //设置提供 展示上下文的代理，在这个上下文中 系统可以展示授权界面给用户
        controller.presentationContextProvider = self;
        //在控制器初始化期间启动授权流
        [controller performRequests];
        //           }
    }
    else {
        // Fallback on earlier versions
    }
}


8. 苹果服务器返回的信息
- (void)authorizationController:(ASAuthorizationController *)controller didCompleteWithAuthorization:(ASAuthorization *)authorization API_AVAILABLE(ios(13.0))
{
    if ([authorization.credential isKindOfClass:[ASAuthorizationAppleIDCredential class]])       {
        ASAuthorizationAppleIDCredential *credential = authorization.credential;
        
        NSString *state = credential.state;
        NSString *userID = credential.user;
        NSPersonNameComponents *fullName = credential.fullName;
        NSString *email = credential.email;
        NSString *authorizationCode = [[NSString alloc] initWithData:credential.authorizationCode encoding:NSUTF8StringEncoding]; // refresh token
        NSString *identityToken = [[NSString alloc] initWithData:credential.identityToken encoding:NSUTF8StringEncoding]; // access token
        ASUserDetectionStatus realUserStatus = credential.realUserStatus;
        NSLog(@"state: %@", state);
        NSLog(@"userID: %@", userID);
        NSLog(@"fullName: %@", fullName);
        NSLog(@"email: %@", email);
        NSLog(@"authorizationCode: %@", authorizationCode);
        NSLog(@"identityToken: %@", identityToken);
        NSLog(@"realUserStatus: %@", @(realUserStatus));
       //这块调后台接口
    }
}


9. 用户调用Apple账户登录的结果
- (void)authorizationController:(ASAuthorizationController *)controller didCompleteWithError:(NSError *)error API_AVAILABLE(ios(13.0))
{
    NSString *errorMsg = nil;
    switch (error.code) {
        case ASAuthorizationErrorCanceled:
            errorMsg = @"用户取消了授权请求";
            break;
        case ASAuthorizationErrorFailed:
            errorMsg = @"授权请求失败";
            break;
        case ASAuthorizationErrorInvalidResponse:
            errorMsg = @"授权请求响应无效";
            break;
        case ASAuthorizationErrorNotHandled:
            errorMsg = @"未能处理授权请求";
            break;
        case ASAuthorizationErrorUnknown:
            errorMsg = @"授权请求失败未知原因";
            break;
    }
    NSLog(@"errorMsg:%@", errorMsg);
}






