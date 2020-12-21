---
title: iOS 5G 网络监测及可能遇到的坑
tags:
  - iOS
  - 开发
categories:
  - iOS
date: 2020-12-21 11:24:28
---

### 前言

 最近在 review 项目时，发现项目上的网络监测没有适配 5G 网络，欲适配之。然在测试过程中发现，缺少在 iOS 14.0.1 的设备上，启动会闪退。debug 发现是 CTRadioAccessTechnologyNRNSA 在 iOS 14.0.1 下不存在，导致闪退。

### Apple 的 CTTelephonyNetworkInfo.h 中的定义

 ```objc
/*
 * Radio Access Technology values
 */
CORETELEPHONY_EXTERN NSString * const CTRadioAccessTechnologyGPRS          API_AVAILABLE(ios(7.0)) API_UNAVAILABLE(macos);
CORETELEPHONY_EXTERN NSString * const CTRadioAccessTechnologyEdge          API_AVAILABLE(ios(7.0)) API_UNAVAILABLE(macos);
CORETELEPHONY_EXTERN NSString * const CTRadioAccessTechnologyWCDMA         API_AVAILABLE(ios(7.0)) API_UNAVAILABLE(macos);
CORETELEPHONY_EXTERN NSString * const CTRadioAccessTechnologyHSDPA         API_AVAILABLE(ios(7.0)) API_UNAVAILABLE(macos);
CORETELEPHONY_EXTERN NSString * const CTRadioAccessTechnologyHSUPA         API_AVAILABLE(ios(7.0)) API_UNAVAILABLE(macos);
CORETELEPHONY_EXTERN NSString * const CTRadioAccessTechnologyCDMA1x        API_AVAILABLE(ios(7.0)) API_UNAVAILABLE(macos);
CORETELEPHONY_EXTERN NSString * const CTRadioAccessTechnologyCDMAEVDORev0  API_AVAILABLE(ios(7.0)) API_UNAVAILABLE(macos);
CORETELEPHONY_EXTERN NSString * const CTRadioAccessTechnologyCDMAEVDORevA  API_AVAILABLE(ios(7.0)) API_UNAVAILABLE(macos);
CORETELEPHONY_EXTERN NSString * const CTRadioAccessTechnologyCDMAEVDORevB  API_AVAILABLE(ios(7.0)) API_UNAVAILABLE(macos);
CORETELEPHONY_EXTERN NSString * const CTRadioAccessTechnologyeHRPD         API_AVAILABLE(ios(7.0)) API_UNAVAILABLE(macos);
CORETELEPHONY_EXTERN NSString * const CTRadioAccessTechnologyLTE           API_AVAILABLE(ios(7.0)) API_UNAVAILABLE(macos);
CORETELEPHONY_EXTERN NSString * const CTRadioAccessTechnologyNRNSA         API_AVAILABLE(ios(14.0)) API_UNAVAILABLE(macos);
CORETELEPHONY_EXTERN NSString * const CTRadioAccessTechnologyNR            API_AVAILABLE(ios(14.0)) API_UNAVAILABLE(macos);
 ```

从 CTTelephonyNetworkInfo.h 中可以发现，5G 网络新增了 CTRadioAccessTechnologyNRNSA、CTRadioAccessTechnologyNR 2 个 value，并且都是 API_AVAILABLE(ios(14.0)) ，即都是从 iOS 14.0 开始可以使用。

```objc
// 故适配 5G 网络只需增加
#ifdef __IPHONE_14_0
        if (@available(iOS 14.0, *)) {
            self.type5G = @[CTRadioAccessTechnologyNRNSA,
                            CTRadioAccessTechnologyNR];
        }
#endif
```

然而，debug 时会发现，在 iOS 14.0.1 的设备在 CTRadioAccessTechnologyNRNSA 报 crash 错误。回想之前 Apple Pay 的适配时出现的问题，将适配代码修改成如下：

```objc
#ifdef __IPHONE_14_1
        if (@available(iOS 14.1, *)) {
            self.type5G = @[CTRadioAccessTechnologyNRNSA,
                            CTRadioAccessTechnologyNR];
        }
#endif
```

修改成 @available(iOS 14.1, *) 后，在 iOS 14.0.1 的设备上运行正常。

之后在网络上搜索发现，我不是第一个遇到这个问题的开发者。参见 [blog](https://blog.csdn.net/myiphon/article/details/109842362) 及回复。

### 思考

自 iOS 13 发布以来，iOS 系统爆发了好几次比较严重的问题。希望 Cook 未来最优可能的继任者 [Craig Federighi](https://www.apple.com.cn/leadership/craig-federighi/) 能扭转这一问题，保持 iOS 系统的研发稳定有序迭代。
