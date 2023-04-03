---
title: iOS中常见Crash总结
date: 2019-02-14 11:33:22
tags: 
    - Crash
categories:
    - iOS
---
> 转自[花丶满楼]<https://juejin.im/post/5c617b85e51d45015e0475ac>

# 1、找不到方法的实现*unrecognized selector sent to instance*
## 1.1、场景对应的Code

```Objective-C
#import "UnrecognizedSelectorVC.h"
/**
 代理协议
 */
@protocol UnrecognizedSelectorVCDelegate <NSObject>
@optional
- (void)notImplementionFunc;
@end
/**
 测试控制器的代理对象
 */
@interface UnrecognizedSelectorVCObj : NSObject<UnrecognizedSelectorVCDelegate>
@property (nonatomic, strong) NSString *name;
@end
@implementation UnrecognizedSelectorVCObj
@end
/**
 测试控制器
 */
@interface UnrecognizedSelectorVC ()
@property(nonatomic, weak) id<UnrecognizedSelectorVCDelegate> delegate;
@property(nonatomic, copy) NSMutableArray *mutableArray;
@end
@implementation UnrecognizedSelectorVC
- (void)viewDidLoad {
    [super viewDidLoad];
    [self case1];
}
/**
 场景一：没有实现代理
 */
- (void)case1 {
    UnrecognizedSelectorVCObj* obj = [[UnrecognizedSelectorVCObj alloc] init];
    self.delegate = obj;
    // 崩溃：reason: '-[UnrecognizedSelectorVCObj notImplementionFunc]: unrecognized selector sent to instance 0x2808047f0'
    [self.delegate notImplementionFunc];
    // 解决办法：应该使用下面的代码
    if ( [self.delegate respondsToSelector:@selector(notImplementionFunc)] ) {
        [self.delegate notImplementionFunc];
    }
}
/**
 场景二：可变属性使用copy修饰
 */
- (void)case2 {
    NSMutableArray* array = [NSMutableArray arrayWithObjects:@1, @2, @3, nil];
    self.mutableArray = array;
    // 崩溃：reason: '-[__NSArrayI addObject:]: unrecognized selector sent to instance 0x281198a50'
    [self.mutableArray addObject:@4];
    // 原因：NSMutableArray经过copy之后变成NSArray
	// @property (nonatomic, copy) NSMutableArray *mArray;
	// 等同于
	// - (void)setMArray:(NSMutableArray *)mArray {
	//    _mArray = mArray.copy;
	//}
    // 解决办法：使用strong修饰或者重写set方法

	// 知识点：集合类对象和非集合类对象的copy与mutableCopy
	// [NSArray copy]                  // 浅复制(新的和原来的是一个array)
	// [NSArray mutableCopy]           // 深复制(array是新的，但是内容还是原来的，内容的指针没有变化)
	// [NSMutableArray copy]           // 深复制(array是新的，但是内容还是原来的，内容的指针没有变化)
	// [NSMutableArray mutableCopy]    // 深复制(array是新的，但是内容还是原来的，内容的指针没有变化)
}
/**
 场景三：低版本系统使用高版本API
 */
- (void)case3 {
    if (@available(iOS 10.0, *)) {
        [NSTimer scheduledTimerWithTimeInterval:1 repeats:YES block:^(NSTimer * _Nonnull timer) {
            
        }];
    } else {
        // Fallback on earlier versions
    }
}
@end
```

## 1.2、原因

找不到方法iOS系统抛出异常崩溃

## 1.3、解决方案：

- [x]  1、实现*消息转发*的几个方法
- [x] 2、尽量避免使用*performSelector*一系列方法
- [x]  3、*delegate* 方法调用前进行 *respondsToSelector* 判断，或者Release模式下使用[ProtocolKit](https://link.juejin.im/?target=https%3A%2F%2Fgithub.com%2Fforkingdog%2FProtocolKit)给协议添加默认实现防止崩溃，Debug模式下关闭默认实现
- [x]  4、*属性*和*成员变量*不要重名定义，合理使用 *synthesize* 生成属性的 *setter* 和 *getter* 方法
- [x]  5、在*MRC*模式下，变量的 *retain* 和 *release* 要谨慎，建议采用安全 *release* 方法，即 *release* 的对象置为 nil
- [x]  6、在.h中声明的方法如果用不到就去掉，用得到就同时在.m文件中实现
- [x]  7、可变属性（如*NSMutableArray*），不要使用*copy*修饰，或者重写set方法
- [x]  8、使用*高版本的系统方法*的时候做判断

## 1.4、知识归纳：参考runtime 消息转发

消息转发机制主要包含三个步骤：

- [x] 1、动态方法解析阶段

```objective-c
+ (BOOL)resolveClassMethod:(SEL)sel 
或者
+ (BOOL)resolveInstanceMethod:(SEL)sel
```

- [x]  2、备用接收者阶段

```objective-c
- (id)forwardingTargetForSelector:(SEL)aSelector
```

- [x]  3、完整消息转发阶段

```objective-c
- (NSMethodSignature *)methodSignatureForSelector:(SEL)aSelector 
和
- (void)forwardInvocation:(NSInvocation *)anInvocation
```




图片摘自：[Objective-C 消息发送与转发机制原理](https://link.juejin.im/?target=http%3A%2F%2Fyulingtianxia.com%2Fblog%2F2016%2F06%2F15%2FObjective-C-Message-Sending-and-Forwarding%2F)

![](/images/upload/168e72062c64f37c.jpg)

> 使用命令：*thread backtrace*查看线程堆栈

# 2、KVC造成的crash
## 2.1、场景对应的Code

```objective-c
#import "KvcCrashVC.h"

@interface KvcCrashVCObj : NSObject
@property (nonatomic, strong) NSString *name;
@end
@implementation KvcCrashVCObj
- (void)setValue:(id)value forUndefinedKey:(NSString *)key {   
}
- (id)valueForUndefinedKey:(NSString *)key {
    return nil;
}
@end

@interface KvcCrashVC ()
@end
@implementation KvcCrashVC
- (void)viewDidLoad {
    [super viewDidLoad];
    [self case1];
}
/**
 场景一：对象不支持KVC
 */
- (void)case1 {
    // reason: '[<NSObject 0x282fe7f90> setValue:forUndefinedKey:]: this class is not key value coding-compliant for the key key.'
    NSObject* obj = [[NSObject alloc]init];
    [obj setValue:@"value" forKey:@"key"];
}
/**
 场景二：key为nil
 */
- (void)case2 {
    // reason: '*** -[KvcCrashVCObj setValue:forKey:]: attempt to set a value for a nil key'
    KvcCrashVCObj* obj = [[KvcCrashVCObj alloc]init];
    // value 为nil不会崩溃
    [obj setValue:nil forKey:@"name"];
    // key为nil会崩溃（直接写nil编译器会提示警告，更多时候我们传的是变量）
    [obj setValue:@"value" forKey:nil];
}
/**
 场景三：key不是object的属性产生的crash
 */
- (void)case3 {
    // reason: '[<KvcCrashVCObj 0x2810bfa80> setValue:forUndefinedKey:]: this class is not key value coding-compliant for the key falseKey.'
    KvcCrashVCObj* obj = [[KvcCrashVCObj alloc]init];
    [obj setValue:nil forKey:@"falseKey"];
}
@end
```



## 2.2、原因

> 给不存在的key（包括key为nil）设置value
>
> - [x]  [obj setValue:@"value" forKey:@"UndefinedKey"]
> - [x]  [obj valueForKey:@"UndefinedKey"]

## 2.3、场景：

### 2.4、解决方案：

- [x]  1、如果属性存在，利用iOS的反射机制来规避，NSStringFromSelector(@selector())将SEL反射为字符串作为key。这样在@selector()中传入方法名的过程中，编译器会有合法性检查，如果方法不存在或未实现会报黄色警告。
- [x] 2、重写类的setValue:forUndefinedKey:和valueForUndefinedKey:

```objective-c
-(void)setValue:(id)value forUndefinedKey:(NSString *)key{

}
-(id)valueForUndefinedKey:(NSString *)key{
    return nil;
}
```

## 2.5、知识归纳：参考runtime kvc原理分析

# 3、EXC_BAD_ACCESS

> 经过ARC的洗礼之后，普通的访问释放对象产生的EXC_BAD_ACCESS已经大量减少了，现在出现的EXC_BAD_ACCESS有很大一部分来自malloc的对象或者越界访问。
>

## 3.1、场景对应的Code

```objective-c
#import "BadAccessCrashVC.h"
#import <objc/runtime.h>
@interface BadAccessCrashVC (AssociatedObject)
@property (nonatomic, strong) UIView *associateView;
@end
@implementation BadAccessCrashVC (AssociatedObject)
- (void)setAssociateView:(UIView *)associateView {
    objc_setAssociatedObject(self, @selector(associateView), associateView, OBJC_ASSOCIATION_ASSIGN);
    //objc_setAssociatedObject(self, @selector(associateView), associateView, OBJC_ASSOCIATION_RETAIN_NONATOMIC);
}
- (UIView *)associateView {
    return objc_getAssociatedObject(self, _cmd);;
}
@end

@interface BadAccessCrashVC ()
@property (nonatomic, copy)                         void(^blcok)(void);
@property (nonatomic, weak) UIView*                 weakView;
@property (nonatomic, unsafe_unretained) UIView*    unSafeView;
@property (nonatomic, assign) UIView*               assignView;
@end
@implementation BadAccessCrashVC
- (void)viewDidLoad {
    [super viewDidLoad];
    [self case1];
}
/**
 悬挂指针：访问没有实现的blcok
 */
- (void)case1 {
    self.blcok();
}
/**
 悬挂指针：对象没有被初始化
 */
- (void)case2 {
    UIView* view = [UIView alloc];
    view.backgroundColor = [UIColor blackColor];
    [self.view addSubview:view];
}
/**
 悬挂指针：访问的对象已经被释放掉
 */
- (void)case3 {
    {
        UIView* view = [[UIView alloc]init];
        view.backgroundColor = [UIColor blackColor];
        self.weakView = view;
        self.unSafeView = view;
        self.assignView = view;
        self.associateView = view;
    }
    // ARC下weak对象释放后会自动置nil，因此下面的代码不会崩溃
    [self.view addSubview:self.weakView];
    // 野指针场景一：unsafe_unretained修饰的对象释放后，不会自动置nil，变成野指针，因此下面的代码会崩溃
    [self.view addSubview:self.unSafeView];
    // 野指针场景二：应该使用strong/weak修饰的对象，却错误的使用assign修饰，释放后不会自动置nil
    [self.view addSubview:self.assignView];
    // 野指针场景三：给类添加添加关联变量的时候，类似场景二，应该使用OBJC_ASSOCIATION_RETAIN_NONATOMIC修饰，却错误使用OBJC_ASSOCIATION_ASSIGN
    [self.view addSubview:self.associateView];
}
@end
```



## 3.2、原因

> 出现悬挂指针，对象没有被初始化，或者访问的对象被释放
>

## 3.3、解决方案：

- [x]  1、Debug阶段开启僵尸模式，Release时关闭僵尸模式
- [x]  2、使用Xcode的Address Sanitizer检查地址访问越界
- [x]  3、创建对象的时候记得初始化
- [x]  4、对象的属性使用正确的修饰方式（strong/weak）
- [x]  5、调用block的时候，做判断

# 4、KVO引起的崩溃

## 4.1、场景对应的Code

```objective-c
#import "KvoCrashVC.h"

@interface KvoCrashVCObj : NSObject
@property (nonatomic, strong) NSString *name;
@end
@implementation KvoCrashVCObj
@end

@interface KvoCrashVC ()
@property (nonatomic, strong) KvoCrashVCObj *sObj;
@end
@implementation KvoCrashVC
- (void)viewDidLoad {
    [super viewDidLoad];
    self.sObj = [[KvoCrashVCObj alloc] init];
//#import <XXShield/XXShield.h>
//    static dispatch_once_t onceToken;
//    dispatch_once(&onceToken, ^{
//        [XXShieldSDK registerStabilityWithAbility:(EXXShieldTypeKVO)];
//    });
}
- (void)touchesBegan:(NSSet<UITouch *> *)touches withEvent:(UIEvent *)event {
    [self func4];
}
/**
 观察者是局部变量，会崩溃
 */
- (void)func1 {
    // 崩溃日志：An -observeValueForKeyPath:ofObject:change:context: message was received but not handled.
    KvoCrashVCObj* obj = [[KvoCrashVCObj alloc] init];
    [self addObserver:obj
           forKeyPath:@"view"
              options:NSKeyValueObservingOptionNew
              context:nil];
    self.view = [[UIView alloc] init];
}
/**
 被观察者是局部变量，会崩溃
 */
- (void)func2 {
    // 崩溃日志：An -observeValueForKeyPath:ofObject:change:context: message was received but not handled.
    KvoCrashVCObj* obj = [[KvoCrashVCObj alloc] init];
    [obj addObserver:self
          forKeyPath:@"name"
             options:NSKeyValueObservingOptionNew
             context:nil];
    obj.name = @"";
}
/**
 没有实现observeValueForKeyPath:ofObject:changecontext:方法:，会崩溃
 */
- (void)func3 {
    // 崩溃日志：An -observeValueForKeyPath:ofObject:change:context: message was received but not handled.
    [self.sObj addObserver:self forKeyPath:@"name" options:NSKeyValueObservingOptionNew context:NULL];
    self.sObj.name = @"0";
}
/**
 重复移除观察者，会崩溃
 */
- (void)func4 {
    // 崩溃日志：because it is not registered as an observer
    [self.sObj addObserver:self forKeyPath:@"name" options:NSKeyValueObservingOptionNew context:NULL];
    self.sObj.name = @"0";
    [self.sObj removeObserver:self forKeyPath:@"name"];
    [self.sObj removeObserver:self forKeyPath:@"name"];
}
/**
 重复添加观察者，不会崩溃，但是添加多少次，一次改变就会被观察多少次
 */
- (void)func5 {
    [self.sObj addObserver:self forKeyPath:@"name" options:NSKeyValueObservingOptionNew context:NULL];
    self.sObj.name = @"0";
}
- (void)observeValueForKeyPath:(NSString *)keyPath ofObject:(id)object change:(NSDictionary<NSKeyValueChangeKey,id> *)change context:(void *)context {
    NSLog(@"keyPath = %@", keyPath);
}
// 总结：KVO有两种崩溃
// 1、because it is not registered as an observer
// 2、An -observeValueForKeyPath:ofObject:change:context: message was received but not handled.
@end
```



## 4.2、原因

> 添加了观察者，没有在正确的时机移除
>

## 4.3、解决方案：

- [x]  1、addObserver和removeObserver一定要成对出现
- [x]  2、推荐使用FaceBook开源的第三方库 FBKVOController

# 5、集合类相关崩溃

## 5.1、场景对应的Code

```objective-c
#import "CollectionCrashVC.h"

@interface CollectionCrashVC ()
@end
@implementation CollectionCrashVC
- (void)viewDidLoad {
    [super viewDidLoad];
    [self case4];
}
/**
 场景一：数组越界
 */
- (void)case1 {
    // reason: '*** -[__NSArrayI objectAtIndex:]: index 4 beyond bounds [0 .. 2]'
    NSArray* array = [[NSArray alloc]initWithObjects:@1, @2, @3, nil];
    NSNumber* number = [array objectAtIndex:4];
    NSLog(@"number = %@", number);
}
/**
 场景二：向数组中添加nil元素
 */
- (void)case2 {
    // reason: '*** -[__NSArrayM insertObject:atIndex:]: object cannot be nil'
    NSMutableArray* array = [[NSMutableArray alloc]initWithObjects:@1, @2, @3, nil];
    [array addObject:nil];
}
/**
 场景三：数组遍历的时候使用错误的方式移除元素
 */
- (void)case3 {
    NSMutableArray<NSNumber*>* array = [NSMutableArray array];
    [array addObject:@1];
    [array addObject:@2];
    [array addObject:@3];
    [array addObject:@4];
    // 不崩溃
    [array enumerateObjectsUsingBlock:^(NSNumber * _Nonnull obj, NSUInteger idx, BOOL * _Nonnull stop) {
        if (obj.integerValue == 1) {
            [array removeObject:obj];
        }
    }];
    // 崩溃，reason: '*** Collection <__NSArrayM: 0x2829946f0> was mutated while being enumerated.'
    for (NSNumber* obj in array) {
        if (obj.integerValue == 2) {
            [array removeObject:obj];
        }
    }
    //    dispatch_async(dispatch_get_global_queue(0, 0), ^{
    //        self.view.backgroundColor = [UIColor blueColor];
    //    });
}
/**
 场景四：使用setObject:forKey:向字典中添加value为nil的键值对，推荐使用KVC的setValue:nil forKey:
 */
- (void)case4 {
    NSMutableDictionary* dictionary = [NSMutableDictionary dictionary];
    [dictionary setObject:@1 forKey:@1];
    // 不崩溃：value为nil，只会移除key对应的键值对
    [dictionary setValue:nil forKey:@1];
    // 崩溃：reason: '*** -[__NSDictionaryM setObject:forKey:]: object cannot be nil (key: 1)'
    [dictionary setObject:nil forKey:@1];
}
@end
```



## 5.2、原因

> 越界、添加nil、多线程非原子性操作、遍历的同时移除元素
>

## 5.3、场景：

- [x]  1、数组越界，访问下标大于数组的个数
- [x]  2、向数组中添加空数据
- [x]  3、多线程环境中，一个线程在读取，一个线程在移除
- [x]  4、一边遍历数组，一边移除数组中的元素
- [x]  5、多线程中操作可变数组（数组的扩容、访问僵尸对象）

## 5.4、解决方案：

- [x]  1、给集合类添加category重写原来的方法，在内部做判断
- [x]  2、使用Runtime把原来的方法替换成自定义的安全方法
- [x]  3、给NSMutableDictionary添加元素的时候，使用setObject:forKey:向字典中添加value为nil的键值对，推荐使用KVC的setValue:nil forKey:。[mutableDictionary setValue:nil ForKey:@"name"]不会崩溃，只是从字典中移除name键值对。
- [x]  4、因为NSMutableArray、NSMutableDictionary不是线程安全的，所以在多线程环境下要保证读写操作的原子性，使用 加锁 、信号量 、GCD串行队列 、GCD栅栏dispatch_barrier_async、CGD组的dispatch_group_enter和dispatch_group_leave

# 6、多线程中的崩溃

## 6.1、场景对应的Code

```objective-c
#import "ThreadCrashVC.h"

@interface ThreadCrashVC ()
@property (nonatomic, strong) NSMutableArray *array;
@end

@implementation ThreadCrashVC
- (void)viewDidLoad {
    [super viewDidLoad];
}
- (void)touchesBegan:(NSSet<UITouch *> *)touches withEvent:(UIEvent *)event {
    [self case1];
}
/**
 dispatch_group_leave比dispatch_group_enter执行的次数多
 */
- (void)case1 {
    // 崩溃：Thread 1: EXC_BREAKPOINT (code=1, subcode=0x1054f6348)
    dispatch_group_t group = dispatch_group_create();
    dispatch_group_leave(group);
}
/**
 在子线程更新UI
 */
- (void)case2 {
    dispatch_async(dispatch_get_global_queue(0, 0), ^{
        self.view.backgroundColor = [UIColor redColor];
    });
}
/**
 多个线程同时释放一个对象
 */
- (void)case3 {   
    // ==================使用信号量同步后不崩溃==================
    {
        dispatch_semaphore_t semaphore = dispatch_semaphore_create(1);
        __block NSObject *obj = [NSObject new];
        dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_HIGH, 0), ^{
            while (YES) {
                dispatch_semaphore_wait(semaphore, DISPATCH_TIME_FOREVER);
                obj = [NSObject new];
                dispatch_semaphore_signal(semaphore);
            }
        });
        while (YES) {
            dispatch_semaphore_wait(semaphore, DISPATCH_TIME_FOREVER);
            obj = [NSObject new];
            dispatch_semaphore_signal(semaphore);
        }
    }
    // ==================未同步则崩溃==================
    {
        __block NSObject *obj = [NSObject new];
        dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_HIGH, 0), ^{
            while (YES) {
                obj = [NSObject new];
            }
        });
        while (YES) {
            obj = [NSObject new];
        }
    }
}
/**
 多线程中的数组扩容、浅复制
 扩容：数组的地址已经改变，报错was mutated while being enumerated
 浅复制：访问僵尸对象，报错EXC_BAD_ACCESS
 
 // 知识点：集合类对象和非集合类对象的copy与mutableCopy
 // [NSArray copy]                  // 浅复制
 // [NSArray mutableCopy]           // 深复制
 // [NSMutableArray copy]           // 深复制
 // [NSMutableArray mutableCopy]    // 深复制
 
 参考：
 [Swift数组扩容原理](https://bestswifter.com/swiftarrayappend/)
 [戴仓薯](https://juejin.im/post/5a9aa633518825556a71d9f3)
 */
-(void)case4 {
    {
        NSArray *array = @[@[@"a", @"b"], @[@"c", @"d"]];
        NSArray *copyArray = [array copy];
        NSMutableArray *mCopyArray = [array mutableCopy];
        NSLog(@"array = %p，copyArray = %p，mCopyArray = %p", array, copyArray, mCopyArray);
    }
    {
        NSMutableArray *array = [NSMutableArray arrayWithObjects:[NSMutableString stringWithString:@"a"],@"b",@"c",nil];
        NSArray *copyArray = [array copy];
        NSMutableArray *mCopyArray = [array mutableCopy];
        NSLog(@"array = %p，copyArray = %p，mCopyArray = %p", array, copyArray, mCopyArray);
    }
    
    dispatch_queue_t queue1 = dispatch_queue_create("queue1", 0);
    dispatch_queue_t queue2 = dispatch_queue_create("queue2", 0);
    
    NSMutableArray* array = [NSMutableArray array];
    
    dispatch_async(queue1, ^{
        while (true) {
            if (array.count < 10) {
                [array addObject:@(array.count)];
            } else {
                [array removeAllObjects];
            }
        }
    });
    
    dispatch_async(queue2, ^{
        while (true) {
            // case 1：数组扩容
            for (NSNumber* number in array) {
              NSLog(@"%@", number);
            }
            // case 2：数组扩容
            NSArray* immutableArray = array;
            for (NSNumber* number in immutableArray) {
              NSLog(@"%@", number);
            }
            // case 3：数组扩容 [NSArray initWithArray:range:copyItems:]
            NSArray* immutableArray1 = [array copy];
            for (NSNumber* number in immutableArray1) {
                NSLog(@"%@", number);
            }
            // case 4：浅复制访问僵尸对象
            NSArray* immutableArray2 = [array mutableCopy];
            for (NSNumber* number in immutableArray2) {
                NSLog(@"%@", number);
            }
        }
    });
}
@end
```



## 6.2、原因

> 死锁、子线程中更新UI、多个线程同时释放一个对象
>

## 6.3、场景

- [x]  1、在子线程中更新UI
- [x] 2、dispatch_group crash，dispatch_group_leave的次数比dispatch_group_enter次数多。参考：iOS疑难问题排查之深入探究dispatch_group crash
- [x]  3、多线程下非线程安全类的使用，如NSMutableArray、NSMutableDictionary。NSCache是线程安全的。
- [x]  4、数据缓存到磁盘和读取。

## 6.4、解决方案：

多线程遇到需要同步的时候，加锁，添加信号量等进行同步操作。一般多线程发生的Crash，会收到SIGSEGV信号，表明试图访问未分配给自己的内存, 或试图往没有写权限的内存地址写数据。

# 7、Socket长连接，进入后台没有关闭

> 当服务器close一个连接时，若client端接着发数据。根据TCP协议的规定，会收到一个RST响应，client再往这个服务器发送数据时，系统会发出一个SIGPIPE信号给进程，告诉进程这个连接已经断开了，不要再写了。而根据信号的默认处理规则，SIGPIPE信号的默认执行动作是terminate(终止、退出),所以client会退出。
>

长连接socket或重定向管道进入后台，没有关闭导致崩溃的解决办法：

## 7.1、解决方案：

- 方法一：1、切换到后台是，关闭长连接和管道，回到前台重新创建。
- 方法二：2、使用signal(SIGPIPE,SIG_IGN)，将SIGPIP交给系统处理，这么做将SIGPIPE设为SIG_IGN，使客户端不执行默认操作，即不退出。

# 8、Watch Dog超时造成的crash

> 主线程执行耗时操作，导致主线程被卡超过一定时间。一般异常编码是0x8badf00d，表示应用发生watch dog超时而被iOS终止，通常是应用花费太多的时间无法启动、终止或者响应系统事件。
>

## 8.1、解决方案：

> 主线程只负责更新UI和事件响应，将耗时操作（网络请求、数据库读写等）异步放到后台线程执行。
>

# 9、后台返回NSNull导致的崩溃，多见于Java做后台服务器开发语言

## 9.1、场景对应的Code

```objective-c
// reason: '-[NSNull integerValue]: unrecognized selector sent to instance 0x2098f99b0'
NSNull *nullStr = [[NSNull alloc] init];
NSMutableDictionary* dic = [NSMutableDictionary dictionary];
[dic setValue:nullStr forKey:@"key"];
NSNumber* number = [dic valueForKey:@"key"];
```

- NULL：用于普通类型，例如NSInteger
- nil：用于OC对象（除了类这个对象）,给nil对象发送消息不会crash
- Nil：用于Class类型对象的赋值（类是元类的实例，也是对象）
- NSNull：用于OC对象的站位，一般会作为集合中的占位元素，给NSNull对象发送消息会crash的，后台给我们返回的就是NSNull对象

NULL：用于普通类型，例如NSInteger
nil：用于OC对象（除了类这个对象）,给nil对象发送消息不会crash
Nil：用于Class类型对象的赋值（类是元类的实例，也是对象）
NSNull：用于OC对象的站位，一般会作为集合中的占位元素，给NSNull对象发送消息会crash的，后台给我们返回的就是NSNull对象

## 9.2、解决方法

> 利用消息转发。参考：NullSafe。当我们给一个NSNull对象发送消息的话，可能会崩溃（null是有内存的），而发送给nil的话，是不会崩溃的。
>

# 10、在iOS中捕获异常信息

> 崩溃主要是由于 Mach 异常、Objective-C 异常（NSException）引起的，同时对于 Mach 异常，到了 BSD 层会转换为对应的 Signal 信号，那么我们也可以通过捕获信号，来捕获 Crash 事件。针对 NSException 可以通过注册 NSUncaughtExceptionHandler 捕获异常信息。

```objective-c
/**
 一、OC异常处理函数

 @param exception exception
 */
static void uncaught_exception_handler (NSException *exception) {
    // 异常的堆栈信息
    NSArray *stackArray = [exception callStackSymbols];
    // 出现异常的原因
    NSString *reason = [exception reason];
    // 异常名称
    NSString *name = [exception name];
    NSString *exceptionInfo = [NSString stringWithFormat:@"Exception reason：%@\nException name：%@\nException stack：%@",
                               name,
                               reason,
                               stackArray];
    NSLog(@"%@", exceptionInfo);
    NSMutableArray *tmpArr = [NSMutableArray arrayWithArray:stackArray];
    [tmpArr insertObject:reason atIndex:0];
    //保存到本地  --  当然你可以在下次启动的时候，上传这个log
    [exceptionInfo writeToFile:[NSString stringWithFormat:@"%@/Documents/error.log",NSHomeDirectory()]
                    atomically:YES
                      encoding:NSUTF8StringEncoding
                         error:nil];
    
    /**
     *  把异常崩溃信息发送至开发者邮件
     */
    NSString *content = [NSString stringWithFormat:@"==异常错误报告==\nname:%@\nreason:\n%@\ncallStackSymbols:\n%@",
                         name,
                         reason,
                         [stackArray componentsJoinedByString:@"\n"]];
    NSMutableString *mailUrl = [NSMutableString string];
    [mailUrl appendString:@"mailto:test@qq.com"];
    [mailUrl appendString:@"?subject=程序异常崩溃，请配合发送异常报告，谢谢合作！"];
    [mailUrl appendFormat:@"&body=%@", content];
    // 打开地址
    NSCharacterSet *set = [NSCharacterSet URLHostAllowedCharacterSet];
    NSString *mailPath = [mailUrl stringByAddingPercentEncodingWithAllowedCharacters:set];
    //NSString *mailPath = [mailUrl stringByAddingPercentEscapesUsingEncoding:NSUTF8StringEncoding];
    [[UIApplication sharedApplication] openURL:[NSURL URLWithString:mailPath]];
}
/**
 二、Unix标准的signal机制处理函数

 @param sig sig
 */
static void handleSignal( int sig ) {
    
}
- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions {
    // OC层中未被捕获的异常，通过注册NSUncaughtExceptionHandler捕获异常信息
    NSSetUncaughtExceptionHandler(&uncaught_exception_handler);
    // 内存访问错误，重复释放等错误就无能为力了，因为这种错误它抛出的是Signal，所以必须要专门做Signal处理
    // OC中层不能转换的Mach异常，利用Unix标准的signal机制，注册SIGABRT, SIGBUS, SIGSEGV等信号发生时的处理函数。
    signal(SIGSEGV,handleSignal);
    return YES;
}
```


### 参考资料

[大白健康系统--iOS APP运行时Crash自动修复系统](https://link.juejin.im/?target=https%3A%2F%2Fneyoufan.github.io%2F2017%2F01%2F13%2Fios%2FBayMax_HTSafetyGuard%2F)
[iOS中的crash防护（一）unrecognized selector sent to instance](https://link.juejin.im/?target=iOS%25E4%25B8%25AD%25E7%259A%2584crash%25E9%2598%25B2%25E6%258A%25A4%25EF%25BC%2588%25E4%25B8%2580%25EF%25BC%2589unrecognized%2520selector%2520sent%2520to%2520instance)
[小萝莉说Crash（一）：Unrecognized selector sent to instance xxxx](https://link.juejin.im/?target=https%3A%2F%2Fcloud.tencent.com%2Fdeveloper%2Farticle%2F1070342)
[iOS实录14：浅谈iOS Crash（一）](https://link.juejin.im/?target=https%3A%2F%2Fwww.jianshu.com%2Fp%2F3261493e6d9e)
[iOS崩溃异常全局捕获](https://link.juejin.im/?target=iOS%25E5%25B4%25A9%25E6%25BA%2583%25E5%25BC%2582%25E5%25B8%25B8%25E5%2585%25A8%25E5%25B1%2580%25E6%258D%2595%25E8%258E%25B7)
[聊聊dealloc](https://link.juejin.im/?target=https%3A%2F%2Fzhongwuzw.github.io%2F2017%2F09%2F21%2F%25E8%2581%258A%25E8%2581%258Adealloc%2F)
[iOS崩溃crash大解析](https://link.juejin.im/?target=https%3A%2F%2Fwww.zybuluo.com%2Fqidiandasheng%2Fnote%2F342315)


#### GitHub开源实现：

- [x]  [XXShield](https://link.juejin.im/?target=https%3A%2F%2Fgithub.com%2FValiantCat%2FXXShield)
- [x]  [QYCrashProtector](https://link.juejin.im/?target=https%3A%2F%2Fgithub.com%2Fqiyer%2FQYCrashProtector)
- [x]  [JKCrashProtect](https://link.juejin.im/?target=https%3A%2F%2Fgithub.com%2FJackLee18%2FJKCrashProtect)
- [x]  [WOCrashProtector](https://link.juejin.im/?target=https%3A%2F%2Fgithub.com%2FWuou%2FWOCrashProtector)
- [x]  [AvoidCrash](https://link.juejin.im/?target=https%3A%2F%2Fgithub.com%2Fchenfanfang%2FAvoidCrash)
- [x]  [NeverCrash](https://link.juejin.im/?target=https%3A%2F%2Fgithub.com%2Fjseanj%2FNeverCrash)