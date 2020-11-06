---
title: App 启动速度优化
tags:
  - 性能优化
categories:
  - iOS
date: 2020-11-06 11:11:52
---



公司的项目，线上功能逐渐稳定，维持在一个月或者半个月稳定发布一次新的版本，下一步我们需要考虑的就是性能方面的优化了。而启动速度，是我们的项目给用户的第一印象，尤为重要，如果启动速度缓慢，会造成比较严重的用户流失，所以，对启动速度的优化，将会成为我们后期开发工作中不可或缺的一部分。

首先，第一步我们需要知道 APP 启动时都做了什么事，从而在对症下药，探索可行的优化方式。

## APP启动

### 基础概念

对于冷启动和热启动这两个名词，相信你一定不陌生吧。

- 冷启动：是指启动前并没有进程在系统里，需要系统新创建一个进程供 APP 使用的启动情况
- 热启动：和热启动相对应，是APP的进程在系统中，用户重新启动进入 APP 的过程，如果把 APP 进程杀掉，然后立刻重启，也属于热启动因为进程的缓存还在

其实，从用户可感知的维度去看呢，就是用户在手机桌面，点击 APP 图标，到 APP 启动图完全消失，首页第一帧渲染完成的过程。所以，本文我们就只展开说 APP 冷启动的流程和优化。

下面我们先来搞清楚 APP 启动时都做了哪些事儿。

### 启动流程

总的来说 APP 启动主要包括三个阶段：

1. main() 函数执行前
2. main() 函数执行后
3. 首屏渲染完成后

关于 APP 启动时间，主要由 pre-main 和 main 之后的时间组成，即：总的启动时间 = main 之前加载的时间 + main 之后加载的时间。启动时间越接近 400ms 越好，并且最好控制在 20s 以内，不然系统会以为 APP 进入一个死循环，应用进程将会被系统强制杀除。

下面我们继续分别说一说这三个阶段系统都做了什么事情。

#### main() 函数执行前

在 main() 函数执行前，系统主要会做下面几件事：

1. 加载可执行文件( APP 的 .o 文件的集合)，就是 Mach-O 文件，这个在后面会详细介绍
2. 加载动态链接库，进行 rebase 指针调整和 bind 符号绑定
3. 运行时的初始化工作，包括类的注册，category 注册，selector 唯一性的检查等
4. 执行 +load() 方法，attribute((constructor)) 修饰的函数的调用，创建 C++ 静态全局变量

pre-main 的加载时间，可以通过添加环境变量 DYLD_PRINT_STATISTICS 的方式打印出来。

![image-20201029100459519](/uploads/image-20201029100459519.png)

运行的时候就可以打印出详细的 pre-main 的时间

```txt
Total pre-main time: 1.0 seconds (100.0%)
 dylib loading time: 608.26 milliseconds (60.3%)
rebase/binding time:  54.06 milliseconds (5.3%)
    ObjC setup time:  42.88 milliseconds (4.2%)
   initializer time: 302.35 milliseconds (30.0%)
   slowest intializers :
     libSystem.B.dylib :   6.10 milliseconds (0.6%)
  libglInterpose.dylib : 146.15 milliseconds (14.5%)
          AFNetworking :  22.28 milliseconds (2.2%)
             MJRefresh :  33.69 milliseconds (3.3%)
                mzmall :  90.76 milliseconds (9.0%)
```

可以看到 main() 方法之前的时间由 dylib loading ，rebse / binding，ObjC setup 和 initializer 四个耗时的部分组成。

所以，相对应这个阶段，我们可做的优化工作如下：

- 减少动态库加载。每个库本身都有依赖，系统会递归的加载所有依赖的动态库，所以苹果公司建议，尽量使用更少的动态库
- 减少无用的类、分类和方法
- +load() 方法中的内容如非必要，可以放到首屏渲染完成之后再执行，或者放在 +initialize() 方法中执行。因为一个 +load() 方法会带来4毫秒的消耗
- 控制 C++ 静态全局变量的数量

下面先说一说 Mach-O 是个什么？

#### Mach-O

理解启动的过程，学习 Mach-O 是不可或缺的。在没有接触过启动优化的相关工作之前，你可能对 Mach-O 比较陌生，本文就来对 Mach-O 做一个简单的介绍。

简单来说，Mach-O 是 macOS 和 iOS 中的一种独有的二进制格式，就是可执行文件的格式。 Mach-O 格式的文件，基本结构都是由Header，Load commands 和 Data 三部分构成：

![image-20201029100824640](/uploads/image-20201029100824640.png)

使用 MachOView 工具可以帮助我们查看 Mach-O 文件的内容——[MachOView 工具](https://github.com/AnleSu/MachOView)，可以直接点击链接获取修改后的源码编译，或者安装根目录文件夹中的 pkg 包，当时我自己安装的时候，下载的安装包遇到了一些打不开和 crash 的问题，所以这里记录了一份可用的安装包，希望对你有帮助。

![image-20201029101822548](/uploads/image-20201029101822548.png)

下面分别说一下 Mach-O 文件的几个组成部分。

**Header** 是一个结构体可以在苹果开源的内核源码中找到

```C
struct mach_header_64 {
    uint32_t        magic;      // 64位还是32位
    cpu_type_t      cputype;    // CPU 类型，比如 arm 或 X86
    cpu_subtype_t   cpusubtype; // CPU 子类型，比如 armv8
    uint32_t        filetype;   // 文件类型
    uint32_t        ncmds;      // load commands 的数量
    uint32_t        sizeofcmds; // load commands 大小
    uint32_t        flags;      // 标签
    uint32_t        reserved;   // 保留字段
};
```

如上面代码所示，包含了表示是 64 位还是 32 位的 magic、CPU 类型 cputype、CPU 子类型 cpusubtype、文件类型 filetype、描述文件在虚拟内存中逻辑结构和布局的 load commands 数量和大小等文件信息。

其中 filetype 表示当前 Mach-O 文件属于哪种类型。Mach-O 包括以下几种类型：

- OBJECT  指的是 .o 文件或者 .a 文件；
- EXECUTE，指的是 IPA 拆包后的文件；
- DYLIB，指的是 .dylib 或 .framework 文件；
- DYLINKER，指的是动态链接器；
- DSYM，指的是保存有符号信息用于分析闪退信息的文件。

**Load Commands** 用于指定布局和文件的链接属性，符号表的位置，动态连接器的路径等。Load Commands 由多个 Load Command 构成，每一个 Load Command 也是一个结构体，并且不同的 Load Command 对应的结构体结构也不相同，但是所有的 Load Command 都必须存在的信息是指令类型 cmd 和指令大小 cmdsize。

Load Commands 中关于映射的段的详细信息都可以在 MachOView 中查看到，这些是一一对应的：

![image-20201029101852935](/uploads/image-20201029101852935.png)

几乎所有的 Mach-O 文件都包含3个段：\_TEXT、\_DATA和 \_LINKEDIT

- __TEXT 包含 Mach header，被执行的代码和只读常量（如C 字符串）。只读（r-x）。
- __DATA 包含全局变量，静态变量等。可读写（rw-）。
- __LINKEDIT 包含了加载程序的『元数据』，比如函数的名称和地址。只读（r–）

#### 内核流程

加载 Mach-O 文件，内核会 fork 进程，并对进程进行一些基本操作，比如为进程分配虚拟内存、为进程创建主线程、代码签名等。整个过程可以在 [XNU的源码](https://github.com/apple/darwin-xnu) 中查看。

![image-20201029102028190](/uploads/image-20201029102028190.png)

这里就不展开说了，感兴趣的自己去扣扣源码看吧，内核做了一系列的准备以及 dyld 的加载，dyld 启动后，接下来的事情将由 dyld 完成，dyld 在程序进程上运行和程序有着相同的权限，接着就构成了下面的 dyld 的工作流水线：

Load dylibs -> Rebase -> Bind -> ObjC -> Initializers

#### Load dylibs

动态链接库的加载过程主要由 dyld 来完成，dyld 是苹果的动态链接器。 系统先读取 App 的可执行文件（Mach-O 文件），从里面获得dyld 的路径，然后加载 dyld，dyld 去初始化运行环境，开启缓存策略，加载程序相关依赖库(其中也包含我们的可执行文件)，并对这些库进行链接，最后调用每个依赖库的初始化方法，在这一步，runtime 被初始化。当所有依赖库的初始化后，轮到最后一位（程序可执行文件）进行初始化，在这时 runtime 会对项目中所有类进行类结构初始化，然后调用所有的 load 方法。最后 dyld 返回 main 函数地址，main 函数被调用，我们便来到了熟悉的程序入口。

dyld 启动后将会在可执行文件的 Mach Header 上找到类型为 LC_LOAD_DYLIB 的加载指令，查找需要的动态库，因为每个 dylib 也是 Mach-O 镜像文件，需要执行同可执行文件一样类似映射到内存上，验证 Mach header 等等，并且 dylib 会依赖其他的 dylib，所以 dylibs 加载是一个递归的过程。

针对这一步骤的优化有：

1. 减少非系统库的依赖
2. 合并非系统库
3. 检查 CocoaPods 引入库的设置，关闭 use_frameworks! 设置，采用静态引用。

#### Rebase

Rebase，是因为 Mach-O 镜像文件加载到内存中的地址和初始地址不同，所以 dyld 需要 Rebase 去进行指针修正。

#### Binding

因为动态库不编译进程序最终的二进制文件中，而是在运行的时候动态的查找调用函数的地址，调用外部符号进行绑定的过程就称作  Binding，比如我们 objc 代码中需要使用到 NSObject，即符号 \_OBJC_CLASS\_$_NSObject，但是这个符号又不在我们的二进制中，在系统库 Foundation.framework 中，因此就需要 binding 这个操作将对应关系绑定到一起

#### Objc Setup

Objc setup 主要是在 objc_init 完成的，数据修正后将会注册 Objc 的类，如果有分类，还需要将分类定义的方法插入的方法列表中，并且保证每一个 selector 是唯一的.

#### initializers

经过上述的步骤之后最后一步就是进行静态初始化工作，例如 Load 函数，如果有 C++ 的一些初始化构造函数也将会被执行。

### main() 函数执行后

main() 函数执行后，是指从 main() 函数开始，到appDelegate 的 didFinishLaunchingWithOptions 方法里首屏渲染相关方法执行完成的过程。

这个过程主要包括了：

1. 日志、统计
2. 配置 APP 运行需要的环境
3. 第三方 SDK 初始化

代码中，各种各样的初始化工作，全部都放在这个阶段去执行了，导致渲染缓慢。这里我们能做的优化工作是，从功能上梳理出来，到底哪些才是首屏渲染必要的初始化功能，哪些是 APP 启动必要的初始化功能，其他只需要在对应模块功能使用前才需要初始化的工作，分别放在他们对应的合适的位置。

除此之外，使用 instrument 可以帮助我们进行分析 didFinishLaunchingWithOptions 中的耗时操作。

### 首屏渲染完成后

到这个阶段，用户已经看到我们 APP 的首页，这个阶段，所做的事情是其他业务模块的一些初始化工作，也就是 appDelegate 的 didFinishLaunchingWithOptions 方法作用域结束的位置。

所以这个阶段的优化优先级比较低，但是还是要注意，那些会阻塞主线程的操作，以防影响用户后面的操作。

### 优化工作

上面每个阶段都说了对应的可以做的优化工作，这里再具体的总结一下。

#### 功能级别的优化

也许你的项目有很多业务线，每个业务线有分属不同的团队去维护，各个业务线有都有他们各自的初始化工作，可能都放在了启动阶段，倒是启动的滞后，所以这里需要各个团队从功能级别，去梳理出必要的初始化工作，和非必要的工作，从而优化启动速度。

#### 动态库加载方面

因为主要的动态库都是系统的动态库，而系统本身对其都有相应的优化处理，所以我们能做的只有去掉无用的系统动态库，或者一步到位直接删除掉 Link Binary With Libraries 中的所有系统动态库，改为自动 link 系统动态库

![image-20201029102124225](/uploads/image-20201029102124225.png)

#### 方法级别的优化

完成了功能级别的优化后，APP 的启动速度应该已经有了一定程度的缩短，下面我们再继续从方法级别去做优化。

1. 删除无用代码

   上面也说过了一个 +load() 方法就会造成 4 毫秒的消耗，那么当我们的项目日渐壮大，业务线交错的时候，也许已经有了很多无用的冗余代码在里面，我们可以使用 **AppCode** 分析检测项目代码。

   > 因为 AppCode 也会分析 Pods 内部的代码，试了设置了也没用，都会去检测，检测时间非常非常非常的久，所以有个比较取巧的办法就是，先移除 Pods 的代码，单纯的分析工程的代码。

   检测完就需要手动移除无用代码了，但是注意检测结果并不十分准确，比如可能存在一些 runtime 映射的方法，也会被检测到，所以还是要人工再检查一遍，以防误删有用代码。

2. 抽象重复代码
   在 iOS 代码中可能会为同一个类写很多分类方法，由于参与开发同学较多，可能会导致方法重复，但是实际上运行起来只能有一个分类的方法被调用，这取决于哪个分类后被加载，然而编译的二进制代码中，两个方法应该是都存在的，这不仅会增加 app 体积，也会增加启动时间，所以应该杜绝这样的重复问题；

   有很多地方可能是名字不同，但是函数的功能相同，这个不容易被发现，需要大家在写代码的过程中注意；

   又或者两个函数名字比较接近，里面有很多相似的代码，这种情况下可以进行相同的代码的提取。

### 具体实施落地

以下以苏打优选 App 举例。

1. 合并各种定制化功能创建的 Category，减少类数量。
2. 注释 Podfile 文件中的 use_frameworks! 配置，采用静态库方式引入第三方库。
3. 编译线上包时，注释 Podfile 文件中 LookinServer、DoraemonKit、MLeaksFinder 等 Debug 所需的第三方库，及注释 Appdelegate 中的相关代码，减少干扰。
4. 删除冗余代码。
5. 抽象重复代码，可以利用 Xcode 的组合快捷键抽取。

优化前后的数据对比（iPhone6s，5次启动平均值，单位 ms ）：

|       统计项        | 冷启动  | 热启动 |
| :-----------------: | :-----: | :----: |
| dylib loading time  | 699.61  | 189.30 |
| rebase/binding time |  55.01  | 15.53  |
|   ObjC setup time   |  47.26  |  34.8  |
|  initializer time   | 330.40  | 320.42 |
| Total pre-main time | 1132.28 | 560.05 |

|       统计项        | 冷启动 | 热启动 |
| :-----------------: | :----: | :----: |
| dylib loading time  | 39.23  | 47.57  |
| rebase/binding time | 41.28  | 14.25  |
|   ObjC setup time   | 32.38  | 28.83  |
|  initializer time   | 317.41 | 215.59 |
| Total pre-main time | 430.31 | 306.25 |

通过数据对比，能看到优化后启动速度提升明显，启动耗时差不多减半，冷启动提升高达 62%，热启动提升也有 35%。
