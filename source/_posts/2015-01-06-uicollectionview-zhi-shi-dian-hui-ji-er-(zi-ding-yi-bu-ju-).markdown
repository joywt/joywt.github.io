---
layout: post
title: "UICollectionView 知识点汇集二（自定义布局）"
date: 2015-01-06 21:25:14 +0800
comments: true
categories: UICollectionView
description: "UICollectionView 知识点汇集二（自定义布局）"
keywords: UICollectionView,UICollectionViewLayout,CustomCollectionViewLayout
---
### 知识点四（自定义布局 UICollectionViewLayout）

在我们开始构建自定义布局前，先考虑一下是否必要这样做。`UICollectionViewFlowLayout` 类已经提供了一系列的显著特性，优化了效率，可以采用不同的方式实现不同类型的标准布局。只有在以下情况下才使用自定义布局：
 
1. 你想要的布局一点也不像网络或是一个行分割的布局，或者布局必须在多个方向上滚动。
2. 你想要经常改变所有单元格的位置，修改现有的流动布局比创建一个自定义布局麻烦。

从API来看实现自定义布局是不难的。唯一难的地方根据需求计算每个item布局中的位置，当位置信息确定了，再实现集合视图就不难了。首先需要理解它核心布局的过程。在布局过程中，collection view 会调用layout对象的一些特定方法。这些方法用来计算items的位置并向 collection view 提供它所需要的信息。除此之外其他的方法也可能会被调用，但下面的这些方法在布局过程中总是以以下顺序调用：

1. `prepareLayout` 预先计算需要提供的布局信息。
2. `collectionViewContentSize` 根据初步的计算返回整个内容区域的大小
3. `layoutAttributesForElementsInRect:` 返回指定区域中 cell 和 views 的属性。

下图说明如何运用这三个方法来生成布局信息。

![如何生成布局信息](http://7sbygq.com1.z0.glb.clouddn.com/image/collectionView/Figure 1_2.png)

用layout对象调用`invalidateLayout`来开始布局过程，对象会调用`prepareLayout`方法。

	调用`invalidateLayout`不会使布局立即响应，collection view 会检查当前布局是否是最新布局，如果不是则更新。也就是说多次调用`invalidateLayout`方法不会引起布局的重复更新。
	
<!-- More -->	
`UICollectionViewLayoutAttributes` 类对象负责布局的属性，可以被缓存和引用。根据视图的类型选择正确的类方法`layoutAttributesForCellWithIndexPath:`,`layoutAttributesForSupplementaryViewOfKind:withIndexPath:` 和 `layoutAttributesForDecorationViewOfKind:withIndexPath:` 创建并返回属性对象。属性对象设置相关的属性来控制单元格和视图的可视性和外观，至少要设置视图在布局中的位置和尺寸。当布局有重叠的情况时，给 `zIndex` 分配值来确保重叠视图的顺序。如果标准属性类不符合我们的需要，我们可以子类话它并扩展它来储存每个视图的其他信息。当子类化布局属性时，它要求我们实现 `isEqual:` 方法来比较自定义属性，因为集合视图在一些操作中会使用该方法。

	@interface MyCustomAttributes : UICollectionViewLayoutAttributes
	@property (nonatomic) NSArray *children;
	@end
	
	// 实现 isEqual: 方法
	
	- (BOOL)isEqual:(id)object{
	    MyCustomAttributes *otherAttributes = (MyCustomAttributes *)object;
	    if ([self.children isEqualToArray:otherAttributes.children]) {
	        return [super isEqual:object];
	    }
	    return NO;
	}

布局过程的最后一步，集合视图调用 `layoutAttributesForElementsInRect:` 方法，返回视图中可见视图里的单元格的属性信息。它在 `prepareLayout` 方法后被调用，所以 `prepareLayout` 方法需要创建足够的布局信息。 `layoutAttributesForElementsInRect:` 方法实现遵循以下步骤：

1. 迭代由 `prepareLayout `方法生成的数据，访问被缓存的属性或者创建新属性。
2. 检查每个数据项中的frame，判断是否与传递给 `layoutAttributesForElementsInRect: `方法的矩形相交互。
3. 对于每个交互的数据项，添加其对应的 `UICollectionViewLayoutAttributes`属性对象到一个数组中。
4. 返回布局属性对象数组。

当 UICollectionView 需要配置插入和删除单元格或者装饰视图动画时，需要重写以下方法

	layoutAttributesForItemAtIndexPath:
	layoutAttributesForSupplementaryViewOfKind:atIndexPath:
	layoutAttributesForDecorationViewOfKind:atIndexPath: 
	
这些方法返回一个布局信息对象。例如：
	
	- (UICollectionViewLayoutAttributes *)layoutAttributesForItemAtIndexPath:(NSIndexPath *)indexPath{
	    UICollectionViewLayoutAttributes *attributes = [super layoutAttributesForItemAtIndexPath:indexPath];
	    /* struct CATransform3D{
	     CGFloat m11 (x缩放),m12 (y 切变), m13 (旋转),m14 ();
	     CGFloat m21 (x切变),m22 (y 缩放), m23 (),m24();
	     CGFloat m31 (旋转),m32 (), m33 (), m34 (透视效果，要操作的对象要有旋转的角度，否则没有效果。正值/负值都有意义);
	     CGFloat m41 (x平移),m42 (y平移),m43 (z平移),m44 ();
	     }
	     整体比例变换时，也就是m11==m22时，若m33>1，图形整体缩小，若0<m33<1，图形整体放大，若m33<0,发生关于原点的对称等比变换。
	     单设ｍ12或ｍ21的时候是切变效果，当【ｍ12=角度】和【ｍ21=－角度】的时候就是旋转效果了。两个角度值相同。
	     */
	    
	    CATransform3D transform = CATransform3DIdentity;
	    //    transform = CATransform3DRotate(transform, (M_PI/180*40), 0, 1, 0);
	    //    transform.m34 = 0.0005;
	    //
	    transform.m34 = -1/800.;
	    transform = CATransform3DRotate(transform, M_PI_2, -1, 0, 0);
	    transform = CATransform3DScale(transform, .8, .8, .8);
	    attributes.transform3D = transform;
	    return attributes;

	}

最后链接自定义布局到 UICollectionView 上，可以通过故事板和代码。代码如下：

	self.collectionView.collectionViewLayout = [[MyCustomLayout alloc] init];

下面通过一个简单的列子 [ClassHierarchicalTree](https://github.com/joywt/ClassHierarchicalTree.git) 来应用上面的知识，可点击链接下载。 代码示例效果图如下：
![类关系图](http://7sbygq.com1.z0.glb.clouddn.com/image/collectionView/Figure 1_3.jpg)


***
扩展阅读

<https://developer.apple.com/library/ios/documentation/WindowsViews/Conceptual/CollectionViewPGforIOS/CreatingCustomLayouts/CreatingCustomLayouts.html>

<https://developer.apple.com/library/ios/documentation/WindowsViews/Conceptual/CollectionViewPGforIOS/AWorkedExample/AWorkedExample.html>