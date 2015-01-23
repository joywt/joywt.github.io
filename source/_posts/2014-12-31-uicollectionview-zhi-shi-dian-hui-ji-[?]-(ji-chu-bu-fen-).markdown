---
layout: post
title: "UICollectionView 知识点汇集一（基础部分）"
date: 2014-12-31 17:38:53 +0800
comments: true
categories: UICollectionView
description: "UICollectionView 知识点汇集一（基础部分）"
keywords: UICollectionView,UICollectionViewLayout,CustomCollectionViewLayout
---

### Collection View 基础

1. UICollectionView 和 UICollectionViewController。UICollectionView 继承 UIScrollView;
2. 管理内容的两个协议：`UICollectionViewDataSource` 和 `UICollectionViewDelegate`。
3. 内容展示的两个视图：`UICollectionReusableView` 和 `UICollectionViewCell`。
4. Layout 布局。`UICollectionViewLayout` 、`UICollectionViewLayoutAttributes` 定义 `UICollectionReusableView` 和 `UICollectionViewCell` 的组织和布局.布局对象并不实际应用布局信息到相应的 views 上，所以从某种意义上说，布局对象就像另一个数据源（data source），只提供视觉信息而不是项目数据。`UICollectionViewUpdateItem` 当项目数据在 Collection View 中执行插入、删除或移动操作，更新这些变化时，Collection View 创建`UICollectionViewUpdateItem` 对象，并传递给布局对象的 `prepareForCollectionViewUpdates:`方法。不需要自己去创建 `UICollectionViewUpdateItem` 对象。
5. flow layout默认布局：`UICollectionViewFlowLayout` 和 `UICollectionViewDelegateFlowLayout`。

下图展示了 collection view 核心对象之间的关联
![figure 1_1](http://7sbygq.com1.z0.glb.clouddn.com/image/collectionView/Figure 1_1.png)

*Supplementary views 和 decoration views 可以有复杂的路径索引，传统上是两个路径索引但并不只限于两个*

### 知识点一（触发 Collection View data source 协议的行为）
当发生下列行为时，UICollectionView 会请求你提供的 data source：

1. UICollectionView 第一次被展示时。
2. 你给 UICollectionView 重新分配 data source 时。
3. 你显式调用 `reloadData` 方法时
4. UICollectionView 的对象执行 `performBatchUpdates:completion:`执行移动、插入或者删除方法时。、

```
4 示例如下：
[self.collectionView performBatchUpdates:^{
    self.primes = [reverseNewPrimes arrayByAddingObjectsFromArray:self.primes];
    [self.collectionView insertItemsAtIndexPaths:indexPaths];
} completion:^(BOOL finished) {
    if (completion != NULL) {
        completion();
    }
}];
```

### 知识点二（单元格cell 长按展示编辑菜单）

长按cell，collection view 会为cell展示出 cut、copy 和 paste 编辑菜单。要展示出编辑菜单，需要满足下面三个条件：

1. 实现下面三个 delegate 协议方法：

   `collectionView:shouldShowMenuForItemAtIndexPath:`

   `collectionView:canPerformAction:forItemAtIndexPath:withSender:`

   `collectionView:performAction:forItemAtIndexPath:withSender:`
   
2. `collectionView:shouldShowMenuForItemAtIndexPath:`方法返回 YES;
3. `collectionView:canPerformAction:forItemAtIndexPath:withSender:` 方法至少对（cut，copy，paste）中的一个行为返回 YES。

		- (BOOL)collectionView:(UICollectionView *)collectionView
		        canPerformAction:(SEL)action
		        forItemAtIndexPath:(NSIndexPath *)indexPath
		        withSender:(id)sender {
		   // Support only copying and pasting of cells.
		   if ([NSStringFromSelector(action) isEqualToString:@"copy:"]
		      || [NSStringFromSelector(action) isEqualToString:@"paste:"])
		      return YES;
		 
		   // Prevent all other actions.
		   return NO;
		}


### 知识点三（使用手势改变布局）
创建手势并添加到 Collection View 上，通过手势关联的 action 改变布局，最后调用 `invalidateLayout`更新布局。如：

	- (void)handlePinchGesture:(UIPinchGestureRecognizer *)sender {
	    if ([sender numberOfTouches] != 2)
	        return;
	 
	   // Get the pinch points.
	   CGPoint p1 = [sender locationOfTouch:0 inView:[self collectionView]];
	   CGPoint p2 = [sender locationOfTouch:1 inView:[self collectionView]];
	 
	   // Compute the new spread distance.
	    CGFloat xd = p1.x - p2.x;
	    CGFloat yd = p1.y - p2.y;
	    CGFloat distance = sqrt(xd*xd + yd*yd);
	 
	   // Update the custom layout parameter and invalidate.
	   MyCustomLayout* myLayout = (MyCustomLayout*)[[self collectionView] collectionViewLayout];
	   [myLayout updateSpreadDistance:distance];
	   [myLayout invalidateLayout];
	}

*注：小心不要混淆 invalidateLayout 和 reloadData 这两个方法。invalidateLayout只更新布局不会使 Collection View 扔掉其视图上的单元格和字视图。*
 
 UICollectionView 监听单个点击手势事件来执行 highlight 和 select 协议方法，你也可添加自定义手势事件。例如添加双击事件：

    UITapGestureRecognizer* tapGesture = [[UITapGestureRecognizer alloc] initWithTarget:self action:@selector(handleTapGesture:)];
    tapGesture.delaysTouchesBegan = YES;
    tapGesture.numberOfTapsRequired = 1;
    [self.collectionView addGestureRecognizer:tapGesture];

`delaysTouchesBegan`属性设置手势的优先级高于 UICollectionView 单个点击手势的监听。了解更多的自定义手势事件查看[Event Handling Guide for iOS.](https://developer.apple.com/library/ios/documentation/EventHandling/Conceptual/EventHandlingiPhoneOS/Introduction/Introduction.html#//apple_ref/doc/uid/TP40009541)

---
扩展阅读：
<https://developer.apple.com/library/ios/documentation/WindowsViews/Conceptual/CollectionViewPGforIOS/Introduction/Introduction.html>
<https://developer.apple.com/library/ios/documentation/UIKit/Reference/UICollectionView_class/index.html>

