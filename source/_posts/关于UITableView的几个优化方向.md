---
title: 关于UITableView的几个优化方向
date: 2019-03-11 14:57:24
tags:
    - 性能优化
categories:
    - iOS
---
如何优化复杂的UITableView卡顿的问题, 一般可以从以下几个方向着手优化：

1. `cell` 的行高不是固定值，需要计算，则要尽可能缓存行高值，避免重复计算行高。因为 `heightForRowAtIndexPath:`是调用最频繁的方法。

2. 滑动时按需加载，这个在大量图片展示，网络加载的时候很管用!(`SDWebImage` 已经实现异 步加载，配合这条性能杠杠的)。

3. 正确使用 `reuseIdentifier` 来重用 `Cells`

4. 尽量少用或不用透明图层

5. 如果 `Cell` 内现实的内容来自 `web`，使用异步加载，缓存请求结果

6. 减少 `subviews` 的数量

7. 在 `heightForRowAtIndexPath:`中尽量不使用 `cellForRowAtIndexPath:`，如果你需要用到它， 只用一次然后缓存结果

8. 所有的子视图都预先创建，如果不需要显示可以设置 `hidden`，尽量少动态给 `Cell` 添加 `View`

9. 颜色不要使用 `alph`

10. 栅格化

11. `cell` 的 `subViews` 的各级 `opaque` 值要设成 YES，尽量不要包含透明的子 `View` `opaque` 用于辅助绘图系统，表示 `UIView` 是否透明。在不透明的情况下，渲染视图时需要快速 地渲染，以提􏰀高性能。渲染最慢的操作之一是混合`(blending`)。提􏰀高性能的方法是减少混合操 作的次数，其实就是 `GPU` 的不合理使用，这是硬件来完成的(混合操作由 `GPU` 来执行，因为这 个硬件就是用来做混合操作的，当然不只是混合)。 优化混合操作的关键点是在平衡 `CPU` 和 `GPU` 的负载。还有就是 `cell` 的 `layer` 的 `shouldRasterize` 要设成 `YES`。

12. `cell` 异步加载图片以及缓存

13. 异步绘制

- (1)在绘制字符串时，尽可能使用 `drawAtPoint: withFont:`，而不要使用更复杂的 `drawAtPoint:(CGPoint)point forWidth:(CGFloat)width withFont:(UIFont *)font lineBreakMode:(UILineBreakMode)lineBreakMode`; 如果要绘制过长的字符串，建议自己先截 断，然后使用 `drawAtPoint: withFont:`方法绘制。

- (2)在绘制图片时，尽量使用 `drawAtPoint`，而不要使用 `drawInRect`。`drawInRect` 如果在绘 制过程中对图片进行放缩，会特别消耗 `CPU`。
- (3)其实，最快的绘制就是你不要做任何绘制。有时通过 `UIGraphicsBeginImageContextWithOptions()` 或者 `CGBitmapContextCeate()` 创建位图会显 得更有意义，从位图上面抓取图像，并设置为 `CALayer` 的内容。 如果你必须实现 `-drawRect:`，并且你必须绘制大量的东西，这将占用时间。
- (4)如果绘制 `cell` 过程中，需要下载 `cell` 中的图片，建议在绘制 `cell` 一段时间后再开启图 片下载任务。譬如先画一个默认图片，然后在 0.5S 后开始下载本 `cell` 的图片。
- (5)即使下载 `cell` 图片是在子线程中进行，在绘制 `cell` 过程中，也不能开启过多的子线程。 最好只有一个下载图片的子线程在活动。否则也会影响 `UITableViewCell` 的绘制，因而影响了 `UITableViewCell` 的滑动速度。(建议结合使用 `NSOpeartion` 和 `NSOperationQueue` 来下载图片， 如果想尽可能找的下载图片，可以把`[self.queuesetMaxConcurrentOperationCount:4];`)
- (6)最好自己写一个 `cache`，用来缓存 `UITableView` 中的 `UITableViewCell`，这样在整个 `UITableView` 的生命周期里，一个 `cell` 只需绘制一次，并且如果发生内存不足，也可以有效的 释放掉缓存的 `cell`。

14. 不要将 `tableview` 的背景颜色设置成一个图片。这回严重影响 `UITableView` 的滑动速度。我曾经翻过一个错误：`self.tableView.backgroundColor = [UIColorcolorWithPatternImage:[UIImageimageNamed:@"background.png"]];` 通过这种方式设置 `UITableView` 的背景颜色会严重影响 UTIableView 的滑动流畅性。修改成 `self.tableView.backgroundColor = [UIColor clearColor];`之后，`fps` 从 `43` 上升到 `60` 左右。 滑动比较流畅。