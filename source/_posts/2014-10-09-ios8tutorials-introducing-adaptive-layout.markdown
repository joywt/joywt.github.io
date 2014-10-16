---
layout: post
title: "ios8Tutorials-Introducing Adaptive Layout"
date: 2014-10-09 22:30:08 +0800
comments: true
categories: 
---
## 第一章：引入自适应布局

iPhone的美好时光啊，只有一个设备，一个屏幕尺寸和一种分辨率。你可以设计界面上的每一个像素点，并且可以保证全世界其他用户看到的和你在测试设备上看到的是一样的。

设计apps非常简单，你可以通过代码用view的frames制定界面，也可以用Interface Builder把view放到xib中，依靠	Springs 和 Structs 来控制。最困难的事情就是处理屏幕旋转，意味着你需要两个设计。

历史证明美好的时光总是不能持久；苹果推出 iPad.构建布局开始变得笨拙。你有一个选择：创建两个NIBS - 一个 iPhone,一个 iPad。

 ![Smaller icon](http://api.photo.yunpan.360.cn/intf.php?method=Photo.getThumb&qid=375125087&nid=141347171349144041&size=1280_1280&devid=&rtick=1413471713&v=1.0.1&devtype=web&sign=4140a0af140e55906df1651d32b3dd6f&xid=266005)
 或者你也可以用代码判断，例如：
 
 	if (UI_USER_INTERFACE_IDIOM() == UIUserInterfaceIdiomPad) { 
 	...	}
看到这些代码我就后背发凉！这两种解决方案都不优雅，它往往是更容易创建两个完全独立的应用程序，一个对 iPhone ，一个对 iPad。
Auto Layout 紧随其后，让你可以通过定义描述UI元素的大小和位置之间关系的语义来布局，而不是单独的指定每个空间的大小和位置。然而，Auto Layout 非常复杂并难以学习，它并没有像苹果希望的那样得到广泛的推广。
现在要考虑如何适应你现有的应用程序到另一个新设备，另一个屏幕大小。当你发现自己第三次重复相同的设计过程，添加第三个 storyboard 或 NIB 和更多的 switch 语句，你知道这是个可怕的错误。
嗯，程序猿们开心起来吧！iOS8引入自适应布局来解决重复设计的问题。底层非常简单 - 针对个别的形体设计不可扩展，因此走向一个更抽象的基于各种 UI 元素之间关系的布局。新的 storyboards 和 size-classes 让你创建一个单一布局，自动适配任意iOS设备的任意屏幕尺寸和方向，甚至包括还没有设计出来的设备。
![Smaller icon](ios8Image/01_02.png)
这本书的部分讲述iOS8的自适应布局。
* Adaptive Layout:`Size classes`, `trait collections` and `universal storyboards` 让你设计出适配所有设备的布局。你可以在抽象的size classes 方面配置你的布局，以便于它能无缝地转移到当前和未来的设备上。
你将从头创建一个自适应布局的 app，同样可以更新你关于auto layout方面的知识。
![Smaller icon](ios8Image/01_03.png)
现在你将构建一个更新多个城市天气的app - 通过这个过程来增强你的 UIKit 里自适应方面的知识。
![Smaller icon](ios8Image/01_04.png)
* Transition Coordinators:设备方向和旋转方法引入了一个新的范式概念。在自适应的世界里，设备旋转只是简单的边界转换。通过自定义创建一些令人惊叹的旋转效果的示例，你会看到它是如何作用于app的。
学习 Transition Coordinators，你需要一个有色块的帮助app并且添加一些靓丽的旋转动画。
![Smaller icon](ios8Image/01_05.png)
* Adaptive View Controller Hierarchies:iPhone 隆重推出 UISplitViewController。所有层次的视图控制器可以像布局一样自适应周围的环境。现在你可以在不同的配置设备上描绘相同复杂的不同层级的试图控制器。

下面说颜色选择器app，这一节你将增强对这个颜色选择器app的认识，它将帮助你去选择整套的颜色。
![Smaller icon](ios8Image/01_06.png)

* Presentation Controllers:Popovers, alerts and search controllers 比以前更加接近啦。当一个新的控制器在屏幕上展示时，它可以称为“presented”;你可以用演示控制器定制整个过程。

你将通过创建一个测试app 来测试你世界上国家的知识。同时学习如何更新 Popovers, alerts and search controllers
![Smaller icon](ios8Image/01_07.png)
一旦你掌握了这些，下一步就可以创建你自己的演示控制器，用绚丽的动画来实现。
![Smaller icon](ios8Image/01_08.png)

看起来有大量的知识需要吸收，但是自适应布局将在下一代app的设计和可发中起到关键的作用。理解这些知识可以使你构建可以在苹果未发布的Apple上运行的app。还有一个好处，你会发现自适应布局在你的开发过程中有积极的影响。

开始着手一个令人兴奋的app设计吧，一个永远不会重复的设计。还在等什么啊，马上开始吧！
