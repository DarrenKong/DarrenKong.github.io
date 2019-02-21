---
title: Swift vs Objective-C
date: 2019-02-21 11:12:48
tags:
    - Swift
    - Objective-C
categories:
    - iOS 
---
| 原文链接：<https://juejin.im/post/5c653aa6e51d457fbf5dc298>
| 转载自：[在路上重名了啊-掘金](https://juejin.im/user/5a7a8b4af265da4e790ff4b3)

> 面试中经常被问到`Objective-C`与`Swift`的区别，其实区别还是很多的，重点整理一下个人觉得很重要的：**面向协议编程**。

### 一、Objective-C与Swift的异同

#### 1.1、swift和OC的共同点：

-  \- `OC`出现过的绝大多数概念，比如**引用计数**、**ARC**（自动引用计数）、**属性**、**协议**、**接口**、**初始化**、**扩展类**、**命名参数**、**匿名函数**等，在`Swift`中继续有效（可能最多换个术语）。
-  \- `Swift`和`Objective-C`共用一套运行时环境，`Swift`的类型可以桥接到`Objective-C`（下面我简称OC），反之亦然

#### 1.2、swift的优点：

-  \- swift注重安全，`OC`注重灵活
-  \- swift注重面向协议编程、函数式编程、面向对象编程，`OC`注重面向对象编程
-  \- swift注重值类型，`OC`注重指针和引用
-  \- swift是静态类型语言，`OC`是动态类型语言
-  \- swift容易阅读，文件结构和大部分语法简易化，只有.swift文件，结尾不需要分号
-  \- swift中的可选类型，是用于所有数据类型，而不仅仅局限于类。相比于`OC`中的`nil`更加安全和简明
-  \- swift中的[泛型类型](https://link.juejin.im?target=https%3A%2F%2Fblog.bombox.org%2F2018-06-14%2Fswift-generic%2F)更加方便和通用，而非`OC`中只能为集合类型添加泛型
-  \- swift中各种方便快捷的[高阶函数](https://link.juejin.im?target=https%3A%2F%2Fmaojianxiang.github.io%2F2017%2F07%2F04%2FSwift%25E4%25B8%25AD%25E7%259A%2584%25E9%25AB%2598%25E9%2598%25B6%25E5%2587%25BD%25E6%2595%25B0%2F)（`函数式编程`） (**Swift的标准数组支持三个高阶函数：map，filter和reduce,以及map的扩展flatMap**)
-  \- swift新增了两种权限，细化权限。**open > public > internal(默认) > fileprivate > private**
-  \- swift中独有的元组类型(`tuples`)，把多个值组合成复合值。元组内的值可以是任何类型，并不要求是相同类型的。

#### 1.3、swift的不足：

-  \- 版本不稳定
-  \- 公司使用比例不高，使用人数比例偏低
-  \- 有些语法其实可以只规定一种格式，不必这样也行，那样也行。像Go一样禁止一切（Go有点偏激）耍花枪的东西，同一个规范，方便团队合作和阅读他人代码。

[Swift 跟 JavaScript 有什么相同和不同点？](https://link.juejin.im?target=https%3A%2F%2Fwww.zhihu.com%2Fquestion%2F24003100)

[从数据结构角度，Golang和Swift对比，有何优缺点？](https://link.juejin.im?target=https%3A%2F%2Fwww.zhihu.com%2Fquestion%2F27746582)

[iOS——Objective-C与Swift优缺点对比](https://link.juejin.im?target=https%3A%2F%2Ficocos.github.io%2F2017%2F09%2F19%2FiOS%25E2%2580%2594%25E2%2580%2594Objective-C%25E4%25B8%258ESwift%25E4%25BC%2598%25E7%25BC%25BA%25E7%2582%25B9%25E5%25AF%25B9%25E6%25AF%2594%2F)

### 二、swift类（class）和结构体（struct）的区别

| 区别                                 | `class` | `struct` |
| ------------------------------------ | ------- | -------- |
| 定义属性用于存储值                   | 是      | 是       |
| 定义方法用于提供功能                 | 是      | 是       |
| 定义附属脚本用于访问值               | 是      | 是       |
| 定义构造器用于生成初始化值           | 是      | 是       |
| 通过**扩展**以增加**默认实现**的功能 | 是      | 是       |
| **遵守协议**以对某类提供标准功能     | 是      | 是       |
| 是否可以**继承**                     | 是      | 否       |
| 是否可以**引用计数**                 | 是      | 否       |
| 类型转换                             | 是      | 否       |
| 析构方法释放资源                     | 是      | 否       |

#### 2.1 定义

```objective-c
class Class {
    // class definition goes here
}

struct Structure {
    // structure definition goes here
}
```

#### 2.2 值 VS 引用

1、结构体 `struct` 和枚举 `enum` 是值类型，类 `class` 是引用类型。 2、`String`, `Array`和 `Dictionary`都是结构体，因此赋值直接是拷贝，而`NSString`, `NSArray` 和`NSDictionary`则是类，所以是使用引用的方式。 3、`struct` 比 `class` 更“轻量级”，`struct` 分配在栈中，`class` 分配在堆中。

#### 2.3 指针

如果你有 C，C++ 或者 Objective-C 语言的经验，那么你也许会知道这些语言使用指针来引用内存中的地址。一个 Swift 常量或者变量引用一个引用类型的实例与 C 语言中的指针类似，不同的是并不直接指向内存中的某个地址，而且也不要求你使用星号（*）来表明你在创建一个引用。Swift 中这些引用与其它的常量或变量的定义方式相同。

#### 2.4 选择使用类和结构体

> **使用struct：任何情况下，优先考虑使用struct，如果满足不了，再考虑class**

-  \- 比如数据被多线程使用，而且数据没有使用class的必要性，就使用struct
-  \- 希望实例被拷贝时，不收拷贝后的新实例的影响
-  \- 几何图形的大小，可以封装width和height属性，都是Double类型
-  \- 指向连续序列范围的方法，可以封装start和length属性，都是Int类型
-  \- 一个在3D坐标系统的点， 可以封装x, y和z属性，都是Double类型

> 

> **使用class**

-  \- 需要**继承**
-  \- 被**递归调用**的时候（参考链表的实现，`node`选用`class`而不是`struct`）
-  \- 属性数据复杂
-  \- 希望引用而不是拷贝

------

[参考链接1：类和结构体](https://link.juejin.im?target=https%3A%2F%2Fnumbbbbb.gitbooks.io%2F-the-swift-programming-language-%2Fcontent%2Fchapter2%2F09_Classes_and_Structures.html) [参考链接2：官方文档](https://link.juejin.im?target=https%3A%2F%2Fdeveloper.apple.com%2Flibrary%2Fcontent%2Fdocumentation%2FSwift%2FConceptual%2FSwift_Programming_Language%2FClassesAndStructures.html%23%2F%2Fapple_ref%2Fdoc%2Fuid%2FTP40014097-CH13-ID82)

### 三、Objective-C中的protocol与Swift中的protocol的区别

相比于`OC`，`Swift` 可以做到`protocol`协议方法的具体**默认实现**（通过`extension`）相比`多态`更好的实现了**代码复用**，而 `OC` 则不行。

### 四、`面向协议`（`面向接口`）与`面向对象`的区别

> `面向对象`和`面向协议`的的最明显区别是`对抽象数据的使用方式`，面向对象采用的是**继承**，而`面向协议`采用的是**遵守协议**。在`面向协议`设计中，`Apple`建议我们更多的使用 **值类型** （`struct`）而非 **引用类型** （`class`）。[这篇文章](https://link.juejin.im?target=https%3A%2F%2Fchausson.github.io%2F2017%2F03%2F01%2F%25E9%259D%25A2%25E5%2590%2591%25E5%258D%258F%25E8%25AE%25AE-POP-%25E4%25BB%25A5%25E9%259D%25A2%25E5%2590%2591%25E5%25AF%25B9%25E8%25B1%25A1-OOP%2F)中有一个很好的例子说明了`面向协议`比`面向对象`更符合**某些业务需求**。其中有飞机、汽车、自行车三种交通工具（均继承自父类交通工具）；老虎、马三种动物（均继承父类自动物）；在古代马其实也是一种交通工具，但是父类是动物，如果马也有交通工具的功能，则：

------

> 如果采用`面向对象编程`，则需要既要继承动物，还要继承交通工具，但是父类交通工具有些功能马是不需要的。由此可见**继承**，作为**代码复用**的一种方式，**耦合性**还是太强。**事物往往是一系列特质的组合，而不单单是以一脉相承并逐渐扩展的方式构建的。**`以后慢慢会发现面向对象很多时候其实不能很好地对事物进行抽象。`

![img](/uploads/168eb6e344f65ff4.jpg)



> 如果采用`面向协议编程`，马只需要实现出行协议就可以拥有交通工具的功能了。**面向协议就是这样的抽离方式，更好的职责划分，更加具象化，职责更加单一。很明显面向协议的目的是为了降低代码的耦合性**。

![img](/uploads/168eb6e6d02aaef6.jpg)



**总结：**

> `面向协议`相对于`面向对象`来说更具有**可伸缩性**和**可重用性**，并且在编程的过程中更加**模块化**，通过协议以及协议扩展替代一个**庞大的基类**，这在**大规模系统编程**中会有很大的便捷之处。

#### 3.1、`协议`和`协议扩展`比`基类`有三个明显的**优点**：

-  **1、类型可以遵守多个协议但是只有一个基类。** 这意味着类型可以随意遵守任何想要特性的协议，而不需要一个巨大的基类。
-  **2、不需要知道源码就可以使用协议扩展添加功能。这意味着我们可以任意扩展协议，包括swift内置的协议，而不需要修改基类**的**源码**。一般情况下我们会给**特定的类**而非**类的层级**（**继承体系**）添加扩展；但是如果必要，我们依然可以给**基类**添加扩展，使所有的子类**继承**添加的功能。使用协议，所有的**属性**、**方法**和**构造函数**都被定义在遵守协议的类型自身中。这让我们很容易地查看到所有东西是**怎么被定义**和**初始化**的。**我们不需要在类的层级之间来回穿梭以查看所有东西是如何初始化的。忘记设置超类可能没有什么大问题，但是在更复杂**的类型中，忘记合理地设置某个属性可能会导致**意想不到**的行为。
-  **3、协议可以被类、结构体和枚举遵守，而类层级约束为类类型。** **协议**和**协议扩展**可以让我们在更合理的地方使用**值类型**。**引用类型**和**值类型**的一个主要的区别就是类型是如何传递的。当我们传递引用类型(class)的实例时，我们传递的对原实例的引用。这意味着所做的任何更改都会**反射回原实例中**。当我们传递值类型的实例时，我们传递的是对**原实例的一份拷贝**。这意味着所做的任何更改都不会反射回原实例中。使用值类型确保了我们总是得到一个唯一的实例因为我们传递了一份对原实例的拷贝而非对原实例的引用。因此，我们能相信没有代码中的其它部分会意外地修改我们的实例。这在**多线程**环境中尤其有用，其中不同的线程可以**修改数据**并创建**意外地行为**。

#### 3.2、面向对象的特点

##### 优点：

-  \- 封装

> 数据封装、访问控制、隐藏实现细节、类型抽象为类；
>
> 代码以逻辑关系组织到一起，方便阅读；
>
> 高内聚、低耦合的系统结构

-  \- 继承

> 代码重用，继承关系，更符合人类思维

-  \- 多态

> 接口重用，父类指针能够指向子类对象
>
> 当父类的引用指向子类对象时，就发生了向上转型，即把子类类型对象转成了父类类型。
>
> 向上转型的好处是隐藏了子类类型，提高了代码的扩展性。

##### 多态的不足：

-  \- 父类有部分public方法是子类不需要的，也不允许子类覆盖重写
-  \- 父类有一些方法是必须要子类去覆盖重写的，在父类的方法其实也是一个空方法
-  \- 父类的一些方法即便被子类覆盖重写，父类原方法还是要执行的
-  \- 父类的一些方法是可选覆盖重写的，一旦被覆盖重写，则以子类为准

##### 较好的抽象类型应该：

-  \- 更多地支持值类型，同时也支持引用类型
-  \- 更多地支持静态类型关联（编译期），同时也支持动态派发（runtime）
-  \- 结构不庞大不复杂
-  \- 模型可扩展
-  \- 不给模型强制添加数据
-  \- 不给模型增加初始化任务的负担
-  \- 清楚哪些方法该实现哪些方法不需实现

#### 3.3、[OneV's Den](https://link.juejin.im?target=https%3A%2F%2Fonevcat.com%2F2016%2F11%2Fpop-cocoa-1%2F)提到的`面向对象`的三个困境：

##### 1、动态派发的安全性（这应该是OC的困境，在Swift中Xcode是不可能让这种问题编译通过的）

> 在`Objective-C`中下面这段代码编译是不会报警告和错误的

```objective-c
NSObject *v1 = [NSObject new];
NSString *v2 = [NSString new];
NSNumber *v3 = [NSNumber new];
NSArray *array = @[v1, v2, v3];
for (id obj in array) {
    [obj boolValue];
}
```

> 在`Objective-C`中可以借助泛型检查这种潜在的问题，Xocde会提示警告

```objective-c
@protocol SafeProtocol <NSObject>
- (void)func;
@end

@interface SafeObj : NSObject<SafeProtocol>
@end
@implementation SafeObj
- (void)func {
    
}
@end

@interface UnSafeObj : NSObject
@end
@implementation UnSafeObj
@end
```

> `Objective-C`在`Xcode7`中，可以使用带**泛型**的**容器**也可以解决这个问题，但是只是⚠️，程序运行期间仍可能由于此问题导致的**崩溃**

```objective-c
SafeObj *v1 = [[SafeObj alloc] init];
UnSafeObj *v2 = [[UnSafeObj alloc] init];
// 由于v2没有实现协议SafeProtocol，所以此处Xcode会有警告
// Object of type 'UnSafeObj *' is not compatible with array element type 'id<SafeProtocol>'
NSArray<id<SafeProtocol>> *array = @[v1, v2];
for (id obj in array) {
    [obj func];
}
```

> 使用`swift`，必须指定类型，否则不是⚠️，而是❌，所以`swift`在**编译阶段**就可以检查出问题

```swift
// 直接报错，而不是警告
// Cannot convert value of type 'String' to expected argument type 'NSNumber'
var array: [NSNumber] = []
array.append(1)
array.append("a")
```

##### 2、横切关注点

我们很难在**不同的继承体系**中**复用代码**，用行话来讲就是**横切关注点**（`Cross-Cutting Concerns`）。比如下面的关注点`myMethod`，位于两条继承链 **(UIViewController -> ViewCotroller 和 UIViewController -> UITableViewController -> AnotherViewController)** 的横切面上。**面向对象**是一种不错的抽象方式，但是肯定不是最好的方式。它无法描述两个**不同事物**具有某个**相同特性**这一点。在这里，**特性的组合**要比继承更贴切事物的本质。

```objective-c
class ViewCotroller: UIViewController {   
    func myMethod() {
        
    }
}

class AnotherViewController: UITableViewController {
    func myMethod() {
        
    }
}

```

在`面向对象编程`中，针对这种问题的几种解决方案：

-  \- 1、**Copy & Paste**

> 快速，但是这也是坏代码的开头。我们应该尽量避免这种做法。

-  \- 2、**引入 BaseViewController**

> 看起来这是一个稍微靠谱的做法，但是如果不断这么做，会让所谓的 `Base` 很快变成垃圾堆。职责不明确，任何东西都能扔进 `Base`，你完全不知道哪些类走了 `Base`，而这个“超级类”对代码的影响也会不可预估。

-  \- 3、**依赖注入**

> 通过外界传入一个带有 myMethod 的对象，用新的类型来提供这个功能。这是一个稍好的方式，但是引入额外的依赖关系，可能也是我们不太愿意看到的。

-  \- 4、**多继承**

> 当然，`Swift` 是不支持多继承的。不过如果有多继承的话，我们确实可以从多个父类进行继承，并将 `myMethod` 添加到合适的地方。有一些语言选择了支持多继承 (比如 `C++`)，但是它会带来 `OOP` 中另一个著名的问题：**菱形缺陷**。

> **在Swift的面向协议编程中，针对这种问题的解决方案**（**使用协议扩展添加默认实现**）：

```swift
protocol P {
    func myMethod()
}
extension P {
    func myMethod() {
        doWork()
    }
}

extension ViewController: P { }
extension AnotherViewController: P { }

viewController.myMethod()
anotherViewController.myMethod()

```

##### 3、菱形问题

> 多继承中，两个父类实现了相同的方法，**子类无法确定继承哪个父类的此方法**，由于多继承的拓扑结构是一个菱形，所以这个问题有被叫做菱形缺陷（`Diamond Problem`）。

上面的例子中，如果我们有多继承，那么 `ViewController` 和 `AnotherViewController` 的关系可能会是这样的：

![img](/uploads/168eb6ee7e40fa00.jpg)



如果`ViewController` 和`UITableViewController`都实现了`myMethod`方法，则在`AnotherViewController`中就会出现菱形缺陷问题。

> 我们应该着眼于写干净并安全的代码，干净的代码是非常易读和易理解的代码。

### 五、编程实践：基于`protocol`的链表实现

```swift
import UIKit

protocol ChainListAble {
    associatedtype T: Equatable
    // 打印
    var description: String{get}
    // 数量
    var count: Int{get}
    
    /// 插入
    func insertToHead(node: Node<T>)
    func insertToHead(value: T)
    func insertToTail(node: Node<T>)
    func insertToTail(value: T)
    func insert(node: Node<T>, afterNode: Node<T>) -> Bool
    func insert(value: T, afterNode: Node<T>) -> Bool
    func insert(node: Node<T>, beforNode: Node<T>) -> Bool
    func insert(value: T, beforNode: Node<T>) -> Bool
    
    /// 删除(默认第一个符合条件的)
    @discardableResult func delete(node: Node<T>) -> Bool
    @discardableResult func delete(value: T) -> Bool
    @discardableResult func delete(index: Int) -> Bool
    //func delete(fromIndex: Int, toIndex: Int) -> Bool
    //func deleteAll()
    
    /// 查找(默认第一个符合条件的)
    func find(value: T) -> Node<T>?
    func find(index: Int) -> Node<T>?
    
    /// 判断结点是否在链表中
    func isContain(node: Node<T>) -> Bool
}

/// [值类型不能在递归里调用](https://www.codetd.com/article/40263)，因此Node类型只能是class而不是struct
// 有些时候你只能使用类而不能使用结构体，那就是递归里
// struct报错：Value type 'Node' cannot have a stored property that recursively contains it
class Node<T: Equatable> {
    var value: T
    var next: Node?
    
    /// 便利构造方法
    ///
    /// - Parameter value: value
    convenience init(value: T) {
        self.init(value: value, next: nil)
    }
    
    /// 默认指定初始化方法
    ///
    /// - Parameters:
    ///   - value: value
    ///   - next: next
    init(value: T, next: Node?) {
        self.value = value
    }
    
    // 销毁函数
    deinit {
        print("\(self.value) 释放")
    }
}

extension Node {
    /// 返回当前结点到链表尾的长度
    var count: Int {
        var idx: Int = 1
        var node: Node? = self
        while node?.value != nil {
            node = node?.next
            idx = idx + 1
        }
        return idx
    }
}

class SingleChainList: ChainListAble {
    typealias T = String
    // 哨兵结点，不存储数据
    private var dummy: Node = Node.init(value: "")
}
extension SingleChainList {
    var description: String {
        var description: String = ""
        var tempNode = self.dummy
        while let nextNode = tempNode.next {
            description = description + " " + nextNode.value
            tempNode = nextNode
        }
        return description
    }
    var count: Int {
        var count: Int = 0
        var tempNode = self.dummy
        while let nextNode = tempNode.next {
            count = count + 1
            tempNode = nextNode
        }
        return count
    }
    
    /// 在头部插入值
    ///
    /// - Parameter value: value
    func insertToHead(value: T) {
        let node: Node = Node.init(value: value)
        self.insertToHead(node: node)
    }
    /// 在头部插入结点
    ///
    /// - Parameter node: node
    func insertToHead(node: Node<T>) {
        node.next = self.dummy.next
        self.dummy.next = node
    }
    /// 在尾部插入值
    ///
    /// - Parameter value: value
    func insertToTail(value: T) {
        let node: Node = Node.init(value: value)
        self.insertToTail(node: node)
    }
    /// 在尾部插入结点
    ///
    /// - Parameter node: node
    func insertToTail(node: Node<T>) {
        var tailNode: Node = self.dummy
        while let nextNode = tailNode.next {
            tailNode = nextNode
        }
        tailNode.next = node
    }
    /// 在指定结点的后面插入新value
    ///
    /// - Parameters:
    ///   - value: 新值
    ///   - afterNode: 指定结点
    /// - Returns: true or false
    func insert(value: T, afterNode: Node<T>) -> Bool {
        let node: Node = Node.init(value: value)
        return self.insert(node: node, afterNode: afterNode)
    }
    /// 在指定结点的后面插入新结点
    ///
    /// - Parameters:
    ///   - value: 新结点
    ///   - afterNode: 指定结点
    /// - Returns: true or false
    func insert(node: Node<T>, afterNode: Node<T>) -> Bool {
        guard self.isContain(node: afterNode) else {
            return false
        }
        node.next = afterNode.next
        afterNode.next = node
        return true
    }
    /// 在指定结点的前面插入新value(双向链表实现这种插入方式速度比单向链表快)
    ///
    /// - Parameters:
    ///   - value: 新值
    ///   - beforNode: 指定结点
    /// - Returns: true or false
    func insert(value: T, beforNode: Node<T>) -> Bool {
        let node: Node = Node.init(value: value)
        return self.insert(node: node, beforNode: beforNode)
    }
    /// 在指定结点的前面插入新结点(双向链表实现这种插入方式速度比单向链表快)
    ///
    /// - Parameters:
    ///   - node: 新结点
    ///   - beforNode: 指定结点
    /// - Returns: true or false
    func insert(node: Node<T>, beforNode: Node<T>) -> Bool {
        var tempNode: Node = self.dummy
        while let nextNode = tempNode.next {
            if nextNode === beforNode {
                node.next = beforNode
                tempNode.next = node
                return true
            }
            tempNode = nextNode
        }
        return false
    }
    /// 删除指定value的结点
    ///
    /// - Parameter value: value
    /// - Returns: true or false
    func delete(value: T) -> Bool {
        var tempNode: Node = self.dummy
        while let nextNode = tempNode.next {
            // 此处判断 == 是否合理
            if nextNode.value == value {
                tempNode.next = nextNode.next
                return true
            }
            tempNode = nextNode
        }
        return false
    }
    /// 删除指定的结点
    ///
    /// - Parameter node: node
    /// - Returns: true or false
    func delete(node: Node<T>) -> Bool {
        var tempNode = self.dummy
        while let nextNode = tempNode.next {
            if nextNode === node {
                tempNode.next = nextNode.next
                return true
            }
            tempNode = nextNode
        }
        return false
    }
    /// 删除指定下标的结点
    ///
    /// - Parameter index: index
    /// - Returns: true or false
    func delete(index: Int) -> Bool {
        var idx: Int = 0
        var tempNode: Node = self.dummy
        while let nextNode = tempNode.next {
            if index == idx {
                tempNode.next = nextNode.next
                return true
            }
            tempNode = nextNode
            idx = idx + 1
        }
        return false
    }
    
    /// 查找指定值的node
    ///
    /// - Parameter value: value
    /// - Returns: node
    func find(value: T) -> Node<T>? {
        var tempNode = self.dummy
        while let nextNode = tempNode.next {
            if nextNode.value == value {
                return nextNode
            }
            tempNode = nextNode
        }
        return nil
    }
    /// 查找指定下标的结点
    ///
    /// - Parameter index: index
    /// - Returns: node
    func find(index: Int) -> Node<T>? {
        var idx: Int = 0
        var tempNode: Node = self.dummy
        while let nextNode = tempNode.next {
            if index == idx {
                return nextNode
            }
            tempNode = nextNode
            idx = idx + 1
        }
        return nil
    }
    /// 判断给定的链表是否在链表中
    ///
    /// - Parameter node: node
    /// - Returns: true or false
    func isContain(node: Node<T>) -> Bool {
        var tempNode = self.dummy.next
        while tempNode != nil {
            if tempNode === node {
                return true
            }
            tempNode = tempNode?.next
        }
        return false
    }
    /// 单向链表反转：方式一非递归实现
    ///
    /// - Parameter chainList: 源链表
    /// - Returns: 反转后的链表
    func reverseList() {
        var prevNode: Node<String>? = self.dummy.next
        var curNode: Node<String>? = prevNode?.next
        var tempNode: Node<String>? = curNode?.next
        prevNode?.next = nil
        while curNode != nil {            
            tempNode = curNode?.next
            curNode?.next = prevNode
            prevNode = curNode
            curNode = tempNode
        }
        self.dummy.next = prevNode
    }
    /// 单向链表反转：方式二递归实现
    ///
    /// - Parameter chainList: 源链表
    /// - Returns: 反转后的链表
    func reverseListUseRecursion(head: Node<T>?, isFirst: Bool) {
        var tHead = head
        if isFirst {
            tHead = self.dummy.next
        }
        guard let rHead = tHead else { return }
        if rHead.next == nil {
            self.dummy.next = rHead
            return
        }
        else {
            self.reverseListUseRecursion(head:rHead.next, isFirst: false)
            rHead.next?.next = rHead
            rHead.next = nil
        }
    }
}

class LinkedListVC: UIViewController {
    var chainList: SingleChainList = SingleChainList.init()
    override func viewDidLoad() {
        super.viewDidLoad()
        // 初始化链表
        for i in 0..<10 {
            let node: Node = Node.init(value: String(i))
            chainList.insertToTail(node: node)
        }
        // 查找结点
        for i in 0..<12 {
            if let find: Node = chainList.find(index: i) {
                debugPrint("find = \(find.value)")
            }
            else {
                debugPrint("not find idx = \(i)")
            }
        }
        // 删除结点
        if chainList.delete(index: 10) {
            debugPrint("删除 index = \(index)成功")
        }
        else {
            debugPrint("删除 index = \(index)失败")
        }
        // 打印结点value信息
        debugPrint(chainList.description)
        // 打印结点个数
        debugPrint(chainList.count)
        // 单向链表反转
        chainList.reverseList()
        // 打印结点value信息
        debugPrint(chainList.description)
        // 单向链表反转
        chainList.reverseListUseRecursion(head: nil, isFirst: true)
        // 打印结点value信息
        debugPrint(chainList.description)
    }
}
```

### 参考博客

[1. 面向协议编程与 Cocoa 的邂逅 (上)](https://link.juejin.im?target=https%3A%2F%2Fonevcat.com%2F2016%2F11%2Fpop-cocoa-1%2F)

[2. 面向协议编程与 Cocoa 的邂逅 (下)](https://link.juejin.im?target=https%3A%2F%2Fonevcat.com%2F2016%2F12%2Fpop-cocoa-2%2F)

[3. [译\] Swift 面向协议编程入门](https://juejin.im/entry/589439622f301e00693567e5)

[4. 面向协议编程初探](https://link.juejin.im?target=https%3A%2F%2Fzhihaozhang.github.io%2F2018%2F05%2F20%2FProtocolOP%2F)

[5. 面向协议(POP)以面向对象(OOP)](https://link.juejin.im?target=https%3A%2F%2Fchausson.github.io%2F2017%2F03%2F01%2F%25E9%259D%25A2%25E5%2590%2591%25E5%258D%258F%25E8%25AE%25AE-POP-%25E4%25BB%25A5%25E9%259D%25A2%25E5%2590%2591%25E5%25AF%25B9%25E8%25B1%25A1-OOP%2F)

[6. 面向对象编程和面向协议编程](https://link.juejin.im?target=https%3A%2F%2Fblog.csdn.net%2Fpermike%2Farticle%2Fdetails%2F72268533)

[7. 面向协议与面向对象的区别](https://link.juejin.im?target=https%3A%2F%2Fblog.csdn.net%2Fweixin_38485842%2Farticle%2Fdetails%2F78777445)

[8. 面向协议编程的一些思考](https://juejin.im/post/5a9640c36fb9a06333155390)

[9. iOS 面向协议方式封装空白页功能](https://juejin.im/post/5ac8a2f4f265da23870f1578)

[10. iOS 面向协议封装全屏旋转功能](https://juejin.im/post/5b9cd4596fb9a05d09654244)

[11. LXFProtocolTool](https://link.juejin.im?target=https%3A%2F%2Fgithub.com%2Fnbhhcty%2FLXFProtocolTool)

[12. 浅谈Swift和OC的区别](https://link.juejin.im?target=https%3A%2F%2Fwww.cnblogs.com%2FyajunLi%2Fp%2F6862164.html)