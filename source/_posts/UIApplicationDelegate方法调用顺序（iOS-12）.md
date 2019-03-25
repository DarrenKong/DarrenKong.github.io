---
title: UIApplicationDelegate方法调用顺序（iOS 12）
date: 2019-03-25 12:30:00
tags: 
    - iOS
categories:
    - iOS
---

> 本文将作为iOS上响应某些操作调用UIApplicationDelegate方法的顺序的指南。除非另有说明，否则本文内容基于iOS 12.1。

### 正常启动

1. `application(_:willFinishLaunchingWithOptions:)`
2. `application(_:didFinishLaunchingWithOptions:)`
3. `applicationDidBecomeActive`

### 正常打开

1. `applicationWillEnterForeground`
2. `applicationDidBecomeActive`

### 通过URL scheme启动

1. `application(_:willFinishLaunchingWithOptions:)`

   示例：

   ```objc
    {
      UIApplicationLaunchOptionsSourceApplicationKey: "com.apple.mobilesafari",
      UIApplicationLaunchOptionsURLKey: "test://test.com"
    }
   ```

2. `application(_:didFinishLaunchingWithOptions:)`

3. [如果 1 和 2 都返回 true] `application(_:open:options:)`

4. `applicationDidBecomeActive`

### 通过URL scheme打开

1. `applicationWillEnterForeground`
2. `application(_:open:options:)`
3. `applicationDidBecomeActive`

### 通过universal link启动

1. `application(_:willFinishLaunchingWithOptions:)`

   示例：

   ```objc
    {
      UIApplicationLaunchOptionsUserActivityDictionaryKey: {
        UIApplicationLaunchOptionsUserActivityIdentifierKey: "F42C288E-D721-4F5D-9AB5-4FEB1D634D99",
        UIApplicationLaunchOptionsUserActivityKey: "<NSUserActivity: 0x2827146a0>",
        UIApplicationLaunchOptionsUserActivityTypeKey: NSUserActivityTypeBrowsingWeb,
      },
      UIApplicationLaunchOptionsSourceApplicationKey: "com.apple.mobilesafari"
    }
   ```

2. `application(_:didFinishLaunchingWithOptions:)`

3. [如果 1 和 2 都返回 true] `application(_:willContinueUserActivityWithType:)`

4. [如果 1 和 2 都返回 true] `application(_:continue:restorationHandler:)`

5. `applicationDidBecomeActive`

### 通过universal link打开

1. `applicationWillEnterForeground`
2. `application(_:willContinueUserActivityWithType:):`
3. `application(_:continue:restorationHandler:):`
4. `applicationDidBecomeActive`

### 通过推送启动

1. `application(_:willFinishLaunchingWithOptions:)`

   示例：

   ```objc
    {
      UIApplicationLaunchOptionsRemoteNotificationKey: {
        aps = {
          alert = "Testing.. (0)";
          badge = 1;
          sound = default;
        }
      }
    }
   ```

2. `application(_:didFinishLaunchingWithOptions:)`

3. `userNotificationCenter(_:didReceive:withCompletionHandler:)`

4. `applicationDidBecomeActive`

### 通过推送打开

1. `applicationWillEnterForeground`
2. `userNotificationCenter(_:didReceive:withCompletionHandler:)`
3. `applicationDidBecomeActive`

### 通过快捷方式启动

1. `application(_:willFinishLaunchingWithOptions:)`

   示例：

   ```objc
    {
      UIApplicationLaunchOptionsShortcutItemKey: <UIApplicationShortcutItem: 0x283ef6e00; type: com.recoursive.example.shortcutTest, title: Shortcut test>
    }
   ```

2. `application(_:didFinishLaunchingWithOptions:)`

3. [如果 1 和 2 都返回 true] `application(_:performActionFor:completionHandler:)`

4. `applicationDidBecomeActive`

### 通过快捷方式打开

1. `applicationWillEnterForeground`
2. `application(_:performActionFor:completionHandler:)`
3. `applicationDidBecomeActive`