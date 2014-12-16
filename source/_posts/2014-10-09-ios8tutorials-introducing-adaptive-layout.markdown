---
layout: post
title: "ios8Tutorials-Introducing Adaptive Layout"
date: 2014-10-09 22:30:08 +0800
comments: true
categories: 
---

*注释：本篇译自 iOS 8 by Tutorials.pdf.本电子文档附源码费用175元。代码随章节会上传到github上*
## 第一章：引入自适应布局

iPhone的美好时光啊，只有一个设备，一个屏幕尺寸和一种分辨率。你可以设计界面上的每一个像素点，并且可以保证全世界其他用户看到的和你在测试设备上看到的是一样的。

设计apps非常简单，你可以通过代码用view的frames制定界面，也可以用Interface Builder把view放到xib中，依靠	Springs 和 Structs 来控制。最困难的事情就是处理屏幕旋转，意味着你需要两个设计。

历史证明美好的时光总是不能持久；苹果推出 iPad.构建布局开始变得笨拙。你有一个选择：创建两个NIBS - 一个 iPhone,一个 iPad。

 ![Smaller icon](../images/ios8Image/01_01.png)
 
 或者你也可以用代码判断，例如：
 
 	if (UI_USER_INTERFACE_IDIOM() == UIUserInterfaceIdiomPad) { 
 	...	}
看到这些代码我就后背发凉！这两种解决方案都不优雅，它往往是更容易创建两个完全独立的应用程序，一个对 iPhone ，一个对 iPad。
Auto Layout 紧随其后，让你可以通过定义描述UI元素的大小和位置之间关系的语义来布局，而不是单独的指定每个空间的大小和位置。然而，Auto Layout 非常复杂并难以学习，它并没有像苹果希望的那样得到广泛的推广。
现在要考虑如何适应你现有的应用程序到另一个新设备，另一个屏幕大小。当你发现自己第三次重复相同的设计过程，添加第三个 storyboard 或 NIB 和更多的 switch 语句，你知道这是个可怕的错误。
嗯，程序猿们开心起来吧！iOS8引入自适应布局来解决重复设计的问题。底层非常简单 - 针对个别的形体设计不可扩展，因此走向一个更抽象的基于各种 UI 元素之间关系的布局。新的 storyboards 和 size-classes 让你创建一个单一布局，自动适配任意iOS设备的任意屏幕尺寸和方向，甚至包括还没有设计出来的设备。
![Smaller icon](../images/ios8Image/01_02.png)
这本书的部分讲述iOS8的自适应布局。
* Adaptive Layout:`Size classes`, `trait collections` and `universal storyboards` 让你设计出适配所有设备的布局。你可以在抽象的size classes 方面配置你的布局，以便于它能无缝地转移到当前和未来的设备上。
你将从头创建一个自适应布局的 app，同样可以更新你关于auto layout方面的知识。
![Smaller icon](../images/ios8Image/01_03.png)
现在你将构建一个更新多个城市天气的app - 通过这个过程来增强你的 UIKit 里自适应方面的知识。
![Smaller icon](../images/ios8Image/01_04.png)
* Transition Coordinators:设备方向和旋转方法引入了一个新的范式概念。在自适应的世界里，设备旋转只是简单的边界转换。通过自定义创建一些令人惊叹的旋转效果的示例，你会看到它是如何作用于app的。
学习 Transition Coordinators，你需要一个有色块的帮助app并且添加一些靓丽的旋转动画。
![Smaller icon](../images/ios8Image/01_05.png)
* Adaptive View Controller Hierarchies:iPhone 隆重推出 UISplitViewController。所有层次的视图控制器可以像布局一样自适应周围的环境。现在你可以在不同的配置设备上描绘相同复杂的不同层级的试图控制器。

下面说颜色选择器app，这一节你将增强对这个颜色选择器app的认识，它将帮助你去选择整套的颜色。
![Smaller icon](../images/ios8Image/01_06.png)

* Presentation Controllers:Popovers, alerts and search controllers 比以前更加接近啦。当一个新的控制器在屏幕上展示时，它可以称为“presented”;你可以用演示控制器定制整个过程。

你将通过创建一个测试app 来测试你世界上国家的知识。同时学习如何更新 Popovers, alerts and search controllers
![Smaller icon](../images/ios8Image/01_07.png)

一旦你掌握了这些，下一步就可以创建你自己的演示控制器，用绚丽的动画来实现。

![Smaller icon](../images/ios8Image/01_08.png)

看起来有大量的知识需要吸收，但是自适应布局将在下一代app的设计和可发中起到关键的作用。理解这些知识可以使你构建可以在苹果未发布的Apple上运行的app。还有一个好处，你会发现自适应布局在你的开发过程中有积极的影响。

开始着手一个令人兴奋的app设计吧，一个永远不会重复的设计。还在等什么啊，马上开始吧！


## 第二章：开始自适应布局

在ios8中引入自适应布局对于 iOS app 设计师是一个巨大的范式转变。apps 现在有能力去适应个别设备和设备方向的布局。当设计你的app时，你只需创建一个布局，就可以在ios8 的所有设备上工作。

这一章为你介绍自适应布局。你将了解通用的故事版，大小类，布局和字体定制和超有用的预览助理编辑器。

在这一章你将创建一个检点的天气应用程序的用户界面 - 并且完全从头开始构建。如果你对自适应布局没有兴趣，不要担心；本章的第一部分为你提供了一步一步构建用户界面使用自适应布局。你会惊讶于你能完成多少而不需编写一行代码！

### 通用的故事板
通用的故事板是通往自适应布局旅途的第一步。同一个故事板可以被同时运行在ios8的 iPads 和 iPhones 上。 没有必要保证每个设备的故事板同步 - 一个单调乏味的过程可能充满了错误。

打开 Xcode 并选择 **Create a new Xcode project** 像这样：
![Smaller icon](../images/ios8Image/02_01.png)

选择 **iOS\Application\Single View Application**,点击 **Next:**
![Smaller icon](../images/ios8Image/02_02.png)

设置 **Product Name** 为 **AdaptiveWeather,Language** 为 Swift 并且确保 **Devices** 设置为 **Universal**，像下面一样：
![Smaller icon](../images/ios8Image/02_03.png)

打开程序，看一看 **Project Inspector** 这里只有一个故事板文件，像下面截图一样：
![Smaller icon](../images/ios8Image/02_04.png)

**Main.storyboard** 是单一的故事板对所有设备，无论设备的屏幕尺寸。打开故事板你将看到它包含一个 view Controller，但是是一个不熟悉的尺寸：
![Smaller icon](../images/ios8Image/02_05.png)

没错，它现在只是个毛坯！故事板在 Xcode 旧版本中必须匹配目标设备的屏幕大小。用于“一个故事板控制所有”方法这显然是不可能的，所以给故事板一个抽象的大小。**Use Size Class** 选项，在 **File Inspector** 找，为你的项目使用这种新格式；选择故事板，打开 **File Inspector** 你将看到像下面这样的选择框：

![Smaller icon](../images/ios8Image/02_06.png)

这个选项对新的所有的ios8项目默认选中。你可以自行关闭当用新的故事板来更新你的旧项目时。想学习如何为ios8更新旧项目，查看本书第四章，*“自适应视图控制器层次结构”*。

开始，打开 **Main.storyboard**并从 **Object Library**拖拽 **Image View**到视图控制器的帆布上。在**Size Inspector**，设置 **X** 坐标为**150**，**Y** 坐标为 **20**.设置 **Width**为**300**，**Height**为**265**。

下一步，从对象库中拖拽一个 **View**并把它放在图片视图的下面。在**Size Inspector**，设置 **X** 坐标为**150**，**Y** 坐标为 **315**.设置 **Width**为**300**，**Height**为**265**。

选择你刚添加的视图，打开 **Identity Inspector** 并在 **Label** 填写 **TextContainer**。注意 **Document** 窗格可能崩溃 - 如果按下 **Show** 按钮来显示它。它给视图一个名字并使它可以在 **Document Inspector** 更容易被看到。这个视图将持有城市和温度的标签。

![Smaller icon](../images/ios8Image/02_07.png)

通常很难看到视图控制器上的视图，因为它们默认背景色是白色和视图控制器一样。解决这个问题，选择视图控制器的视图，打开 **Attributes Inspector** 设置背景色为 **Red:74,Green:171,Blue:247**。

下一步，选择 TextContainer 视图设置背景色为**Red:55,Green:128,Blue:186**

你的视图控制器将显示像下面一样：

![Smaller icon](../images/ios8Image/02_08.png)

这两个视图是试图控制器上唯一的子视图；现在你的任务是给他们呢添加一些约束。

### 添加布局约束
选择图片视图按下自适应工具上的 **Align** 按钮。检查 **Horizontal center in Container**,确保值设置为 **0**。按下 **Pin** 按钮并添加接近头部的约束为 0，如下：

![Smaller icon](../images/ios8Image/02_09.png)

上面添加的约束确保图片视图和头部有一个确定的尺寸间距并且处于左边到右边的中间。现在你需要配置图片视图和文本容器视图之间的间距。**Ctrl-drag**从图片视图向下到容器视图，像这样：

![Smaller icon](../images/ios8Image/02_10.png)

再一次展示出约束的上下文菜单。选择 **Vetical Spacing** 像下面这样：

![Smaller icon](../images/ios8Image/02_11.png)

这个约束定义了图片视图底部和文本容器视图头部上下间距的量。

**Size Inspector** 在 Xcode 6 中看起来有点不同。选择图片视图打开 **Size Inspector** 看它现在看起来是什么样的：

![Smaller icon](../images/ios8Image/02_12.png)

你将看到你刚添加的三个约束；你可以容易的配置每一个约束从尺寸编辑器的内部。按下 **Buttom Space To:TextContainer** 中的 **Edit** 按钮；你会看到一个对话框来配置约束属性。设置 **Constant** 等于 **20** 像下面：

![Smaller icon](../images/ios8Image/02_13.png)

点击其他地方关闭会话框。

你已经配置了你的文本容器视图和图片视图的底部20点像素的间隙，但是你还需要添加视图的另外三个边的约束。

选择文本容器视图点击下面栏上的 **Pin** 图标显示 **Add New Constraints** 对话框。在 **Spacing to nearest neighbor** 选项上，设置 **left，right，bottom** 间隔为 **0**。确保 **Constrain to margins** 是 **unchecked**；这将移除你在视图上的填充。

![Smaller icon](../images/ios8Image/02_14.png)

点击 **Add 3 constraints** 添加新的约束到你的视图上。这个别针文本容器在视图控制器视图上的左、右、下三边。

你的故事板将看起来像下面截图这样：

![Smaller icon](../images/ios8Image/02_15.png)

你注意到在试图上有一些橙色的约束；标明这写约束存在问题需要你注意。故事板自动更新的框架可以满足包含试图的这些约束，但是如果你这样做了，你的图片视图将缩小为0尺寸。这是因为图片视图没有任何内容 - 这表示它的宽高为0。自动布局依赖于视图的内在大小来确定它的宽高，如果不提供物理的宽高限制。

在 **Prpject Navigator**,打开 **Images.xcassets,** 次看点击左下方的 **+** 号并在弹出菜单中选择 **New Image Set**：

![Smaller icon](../images/ios8Image/02_16.png)

双击详情视图左上方的标题 **Image** 重命名为 **cloud**，像下面一样：

![Smaller icon](../images/ios8Image/02_17.png)

你的新图片默认是一组图片，每一个文件对特定的场景都有一个目的大小。例如，一组图像可能有 non-Retina，Retain 和 Retain HD 版本的相同图像。由于 asset 库继承于 UIKit，设定图像组的名字 iOS 在运行的时候会选取正确的图像。
	
	注意：如果你以前用过 asset 库，你会发先这里有些奇怪 - 3x是什么？这个新的展示比率用于 iPhone 6 Plus 的 Retain HD上。这表明在设备的每个方向上这里一个点有3个像素。3x的图像像素是1x的9倍。
	
你的 cloud 图片是空的。解压这章节里面的 **cloud_images.zip**，里面有5个文件。从文件夹拖拽 **cloud_small.png** 到 **cloud** 的 **1x** 格子中，像这样：

![Smaller icon](../images/ios8Image/02_18.png)

由于图片是白色背景透明，在 asset 库中不容易看到。然而，你可以选中 **1x** 格子按下 **space** 快速预览：

![Smaller icon](../images/ios8Image/02_19.png)

现在拖拽 **cloud_small@2x.png** 到 **2x** 格子中，**cloud_small@3x.png** 到 **3x** 格子中。

现在你可以用图片来填充你的图片视图了。回到 **Main.storyboard** 选择你的图片视图。选择 **Attributes Inspector**，在 **Image** 栏中输入名字 **cloud** ，并在 **View\Mode** 的下拉菜单中选择 **Aspect Fit**，像这样：

![Smaller icon](../images/ios8Image/02_20.png)

你的故事板现在看起来像下面这样：

![Smaller icon](../images/ios8Image/02_21.png)

这是你的图片 - 但是橙色的约束提醒这你还有一点工作要做。首先选择视图控制器的视图，选择底部栏中的 **Resolve Auto Layout Issues** 图标。最后，选择对话框中的 **All Views\Update Frames** 如下：

![Smaller icon](../images/ios8Image/02_22.png)

视图控制器解决不满足约束条件的视图后，你的故事板看起来像这样：

![Smaller icon](../images/ios8Image/02_23.png)
 
### 预览助理编辑

通常你现在会编译运行你的项目在 iPad, iPhone 4s, 5s, 6 and 6 Plusv模拟器上 - 使用不同的方向 - 来测试这个新的通用故事板。这是个艰苦的过程，但是 xcode 6 给你了个更好的选择 - 预览编辑器。

打开 **Main.storyboard** 在菜单栏点击 **Assistant Editor** 按钮：

![Smaller icon](../images/ios8Image/02_24.png)

屏幕分厂了两个窗格。在 **Jump Bar** 点击 **Automatic** 并选择下拉菜单中的 **Preview**。最中选择 **Main.storyboard** ，像这样:

![Smaller icon](../images/ios8Image/02_25.png)

助理编辑器中的故事板现在展示的是 4-inch iPhone 的屏幕，如下：

![Smaller icon](../images/ios8Image/02_26.png)

点击下面预览名称（**iPhone 4-inch**）旁边的面板可预览旋转效果。将展示横屏效果，如下：

![Smaller icon](../images/ios8Image/02_27.png)

这是个巨大的进步对于多个模拟器上 - 等等!这里还有更多！点击左下角的 **+** 按钮：

![Smaller icon](../images/ios8Image/02_28.png)

选择 **iPhone 5.5-inch** 和 **iPad** 添加多个预览如下：

![Smaller icon](../images/ios8Image/02_29.png)

注意横屏预览中的问题了吗？cloud 图片太大了。解决这个需要给图片视图添加新的约束。

回到故事板中。从图片视图 **Ctrl-drag** 到视图控制器视图添加新约束。从上下文菜单中选择 **Equal Heights** 如下：
 
![Smaller icon](../images/ios8Image/02_30.png)

故事板中的一些约束变红了。这是因为刚加的约束影响到了其他约束，作为图像视图不可能和视图控制器视图同一高度并且维护前面创建的垂直边缘。

在 **Document Outline** 中选中刚添加的约束并且打开 **Attributes Inspector**。如果 **First Item** 没有设置为 **cloud.Height** 此刻在下拉中选择 **Reverse First and Second Item** 如下：

![Smaller icon](../images/ios8Image/02_31.png)

下一步，选择 **Relation** 设置为 **Less Than or Equal** 设置 **Multiplier** 为 **0.40** 如下：

![Smaller icon](../images/ios8Image/02_32.png)

这表示cloud 图片会是图片的实际大小或者屏幕高度的40%，哪个小是那个。

你会发现当你更新约束后，预览面板自动更新了。如下：

![Smaller icon](../images/ios8Image/02_33.png)

如果预览面板有多个设备，也会更新。

既然是提供天气的app，你就需要添加一些label 来显示城市名字和当前温度。

### 添加内容到文本容器

从对象资源库中拖拽两个 labels 到 Main.storyboard 的文本容器中，简单排列如下：

![Smaller icon](../images/ios8Image/02_34.png)

选择上面的 label 用 **Align** 和 **Pin** 菜单设置 label 横向居中，并设置头部距最邻近视图的间距为 10 ，如下面所示：

![Smaller icon](../images/ios8Image/02_35.png)

下一步，选择 **Attributes Inspector** 然后设置 **Text** 为 **Cupertino**, **Color** 为 **White**，字体为 **Helvetica Neue，Thin** **Size** 为 150。

你发现文本显示有问题，这是由 lable 的 frame 引起的。然而马上你就会解决它。

现在选择另一个 label，像上一个一样设置它的横向居中，底部距最邻视图为 10。检查它的**Size Inspector** 像下面一样：

![Smaller icon](../images/ios8Image/02_36.png)

用 **Attributes Inspector** 设置 **Text** 为 **28C**, **Color** 为 **White**，字体为 **Helvetica Neue，Thin** **Size** 为 250。

现在你需要解决上面遗留的问题。选择视图控制器的 view，点击故事板的底部 **Resolve Auto Layout Issues** 按钮，然后选择 **All Views\Update Frames**.故事板更新如下：

![Smaller icon](../images/ios8Image/02_37.png)

labels 的文字重叠在了一起，不是你想要的效果。然而，不做任何修改它在 iPad 上显示是正确的：

![Smaller icon](../images/ios8Image/02_38.png)

可见，对于 iPhone 文本的字体太大：

![Smaller icon](../images/ios8Image/02_39.png)

在下一章，你将解决这些大小的问题。

### Size Classes （大小类）

通用故事本是伟大的 - 但是你已经发现所有显示创建一个布局是一种挑战。然而，自适应布局有一些工具和技巧可以解决这些问题。

自适应布局背后的核心概念之一是 **Size Classes**。大小类是一个属性，应用于任意视图和视图控制器上的内容可以显示在给定的水平和垂直维度。Xcode 6提供了两个大小类：**Regular** 和 **Compact**.尽管它们和视图的物理维度有关，但是它们也代表了视图的语义上的大小。

下表展示了大小类在不同设备和方向上是怎样的：

|            |Vertical Size Class | Horizontal Size Class |
|------------|--------------------| --------------------- |
|iPad Portrait| Regular | Regular |
|iPad Landscape|Regular|Regular|
|iPhone Portrait|Regular|Compact|
|iPhone Landscape|Compact|Compact|
|iphone 6 Plus Landscape|Compact|Regular|

这是大小类对应用程序的设置手册。然而，你可以在视图层次结构的任意一点重写这些大小类。这可能是有用的当使用视图控制器在一个远远小于屏幕的容器里 - 你将在自适应布局的中间章节看到这样一个实例。

对于你和你的应用程序设计这意味这什么？虽然你的应用程序识别大小类，你构建的布局是大小类无关的 - 也就是说，你的布局在所有的大小类里面是一样的。

在设计自适应适配的阶段这是一个很重要的点。首先你需要建立一个基本布局，然后在基于个人需求的大小类上自定义每个特定的大小类。不要把每个大小类作为完全独立的设计。把自适应布局像层级结构一样考虑，把所有公共的设计放在父级层次，在子级层次中做必要的修改。

这里几乎没有提到特定设备的配置布局。这是因为自适应布局的的核心概念是大小类脱离特定设备的特点。这意味着一个支持自适应布局的视图，可用于全屏视图控制器以及包含视图控制器，即使不同的外观。

这同样有利于苹果，在设备扩大空间范围和特点的同时没有迫使开发人员和设计人员重新设计他们的应用程序。


### Working with class sizes

点击 Interface Builder 底部的 **w Any h Any**；你将看到大小类选择器如下：

![Smaller icon](../images/ios8Image/02_40.png)

在这里你可以选择一个大小类显示在故事板中，通过网格中的单元格。这里有9个选项：3种对应垂直视图（any，regular or compact）3种对应水平视图。总数是 3x3=9 种组合。
	
	注意：在命名这点上有轻微差异。大小类中称为 horizontal and vertical - 你将在下一章学习关于底层类，UITraitCollection。然而，IB 中使用 width 和 height。（width = horizontal;height = vertical）,这里只需知道有两个平等的概念。
	
你目前的布局在紧凑的高度中有问题。为了解决它，选择 **Any Width | Compact Height** 大小类像下面一样：

![Smaller icon](../images/ios8Image/02_41.png)

立即发现在编辑器中有两个不同：

![Smaller icon](../images/ios8Image/02_42.png)

1.帆布的形状变化代表新的大小类

2.底部栏变得更蓝。这标明你现在正在进行具体布局。

为了改变布局，你需要暂时改变一些约束。在自适应术语中称为 **installing** 和 **uninstalling** 约束。安装一个约束如果是当前活动，而卸载约束不是当前活动在当前的大小类。

选择图片视图，然后打开 **Size Inspector**。你可以看到所有影响这个视图的约束：

![Smaller icon](../images/ios8Image/02_43.png)

点击选择 **Align Center X to: Superview** ，然后按下 **Delete** 卸载这个约束在当前的大小类中。这个约束立即从故事板中消失并且变成灰色：

![Smaller icon](../images/ios8Image/02_44.png)

	注意：你需要切换 **This Size Class** 为 **All**，才能看到这个卸载的约束。
	
双击卸载的约束来选择这个约束。底部有一个额外的行，如下：

![Smaller icon](../images/ios8Image/02_45.png)

这表明这个约束安装在基本布局中，但是不作用与 **Any Width | Compact Height**布局 - 也就是，当前编辑的这个布局。

重复同样的操作来卸载其他左右与图片视图上的三个约束。你的文档大纲和图片视图的尺寸编辑器看起来如下图：

![Smaller icon](../images/ios8Image/02_46.png)

现在你可以为这个大小类添加所需要的约束。用 **Align** 和 **Pin** 菜单设置纵向居中，并设置左边距最邻近视图的间距为 10：

![Smaller icon](../images/ios8Image/02_47.png)

从图片视图 **Ctrl-drag** 到试图控制器视图，然后在弹出菜单中选择 **Equal Widths**。

打开图片视图的尺寸编辑器，然后双击 **Equal Width to：Superview** 约束。如果 **First Item** 不是 **cloud.width** ,用下拉菜单 **Reverse First and Second Item** ，然后更新 **Multiplier** 为 0.45。

现在图片视图的所有大小类中的约束设置完成，但是文本容器还需要一些关注。你需要修改约束移动label 到右边。

**TextContainer** 视图内部约束位置标签还保持原样。但是现有的三个约束 - 视图的左、右、下三边 - 存在问题。为了使视图在父视图的右下边，你需要卸载左手边的约束。

从文档大纲或者故事板中选择左手边约束，如下所示：

![Smaller icon](../images/ios8Image/02_48.png)

按下**Cmd-Delete**卸载这个约束。和前面一样，任何卸载的约束在文档大纲中出现灰色的轮廓。

为了文本容器的在正确的位置上还需要添加两个约束。这个视图只有父视图一半的宽度，并头部固定。理论上，你可以想以前一样简单从文本容器视图拖拽到视图控制器视图。然而，使用文本大纲来做更容易。

在文本大纲中从图片视图 **Ctrl-drag** 到试图控制器视图，如下：

![Smaller icon](../images/ios8Image/02_49.png)

菜单显示在选中的约束旁边。Shift-click Top Space to Top Layout Guide and Equal Widths 如下：

![Smaller icon](../images/ios8Image/02_50.png)

打开 **Size Inspector** 更新 **TextContainer** 上你刚添加的两个约束如下：

1.**Top Space to: Top Layout Guide** 设置 **Constant** 为 0

2.**Equal Width to: Superview** 设置 **Multiplier** 为 0.5.请注意,您可能需要切换第一和第二项在这个约束,正如您之前所做的。双击约束，选择 **Reverse First and Second Item.**。

点击故事板的底部 **Resolve Auto Layout Issues** 按钮，然后选择 **All Views\Update Frames**.故事板更新如下：

![Smaller icon](../images/ios8Image/02_51.png)

布局改变完成。你马上就完成了。这里还有一点字体大小问题 - 你会在下一节解决这些。



 