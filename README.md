# LocalAuthentication
指纹验证接口demo
导入头文件<LocalAuthentication/LocalAuthentication.h>就可以用, 核心代码如下:

    LAContext *context = [[LAContext alloc] init];
    NSError *error = nil;
    
    // context.localizedFallbackTitle = @""; // 如需移除输入密码按钮, 必须设置该字符串为空
    
    // 判断是否能验证指纹
    if ([context canEvaluatePolicy:LAPolicyDeviceOwnerAuthenticationWithBiometrics error:&error]) {
        
        // 判断验证指纹的结果
        [context evaluatePolicy:LAPolicyDeviceOwnerAuthenticationWithBiometrics localizedReason:NSLocalizedString(@"使用指纹登录", nil) reply:^(BOOL success, NSError *error) {
            if (success) {
                NSLog(@"登录成功");
                dispatch_async(dispatch_get_main_queue(), ^{
                    NSLog(@"更新UI");
                    // 所有回调都在异步, 如需更新UI, 要回到主线程, 后面也一样(否则会经历非常漫长的过程才能更新成功)
                });
            } else {
                switch (error.code) {
                    case LAErrorUserCancel:
                        NSLog(@"手动取消了验证"); // 用户点击了取消按钮时调用
                        break;
                        
                    case LAErrorAuthenticationFailed: // 验证失败时调用(连续3次验证没通过)
                        NSLog(@"验证失败");
                        break;
                        
                    case LAErrorSystemCancel:
                        NSLog(@"验证被系统取消"); // 当锁频,接入电话等情况会调用
                        break;
                        
                    case LAErrorUserFallback:
                        NSLog(@"使用密码登录"); // 用户点击了使用密码按钮时调用, 密码登录模块需自己实现
                        break;
                    default:
                        break;
                }
            }
        }];
    } else {
        switch (error.code) {
            case LAErrorPasscodeNotSet:
                NSLog(@"没有设置密码"); // 登记指纹一定要设置密码, 所以当没有设置密码时, 无法进行指纹验证
                break;
                
            case LAErrorTouchIDNotEnrolled:
                NSLog(@"没有登记指纹"); // 没有登记指纹时, 无法进行指纹验证
                break;
                
            case LAErrorTouchIDNotAvailable:
                NSLog(@"设备不支持指纹"); // 设备不支持指纹时, 无法进行指纹验证
                break;
            default:
                break;
        }
    }

