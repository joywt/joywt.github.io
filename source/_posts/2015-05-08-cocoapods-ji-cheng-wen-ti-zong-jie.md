---
layout: post
title: "CocoaPods 集成问题总结"
date: 2015-05-08 09:39:00 +0800
comments: true
categories: cocoaPods
keywords: Undefined symbols for architecture,objc-class-ref,cocoaPods
---
由于项目的越来越庞大，团队成员的流动变化，如何让新同事快速的了解项目结构就变得非常必要了。为了解决这个问题，我们准备使用 cocoapods 来管理第三方库并用文档维护。这样就需要删除项目中的原来第三方库，然后通过 cocoapods 重新引入。
问题一
---
通过 cocoapods 引入 AFNetworking,执行 pod install 终端提示如下：

```
[!] The `YYW [Debug]` target overrides the `OTHER_LDFLAGS` build setting defined in `Pods/Target Support Files/Pods/Pods.debug.xcconfig'. This can lead to problems with the CocoaPods installation
    - Use the `$(inherited)` flag, or
    - Remove the build settings from the target.

[!] The `YYW [Release]` target overrides the `OTHER_LDFLAGS` build setting defined in `Pods/Target Support Files/Pods/Pods.release.xcconfig'. This can lead to problems with the CocoaPods installation
    - Use the `$(inherited)` flag, or
    - Remove the build settings from the target.
```
解决办法： 

* 选择程序 **xxx -> TARGETS -> Build Settings** 
+ 找到 **Other Linker Flags** 如果里面有配置参数先删除，添加 **$(inherited)**
+ 然后执行 **pod update**

问题二
---
编译程序提示错误：**Undefined symbols for architecture i386** 或 **Undefined symbols for architecture armv7**

解决方法：

* 编辑 **Podfile**
* 添加

```
post_install do |installer_representation|
    installer_representation.project.targets.each do |target|
        target.build_configurations.each do |config|
            config.build_settings['ARCHS'] = 'armv7 armv7s'
        end
    end
end  
```
- 注意 **config.build_settings['ARCHS'] = 'armv7 armv7s'** 和项目的 **xxx -> TARGETS -> Build Settings 中的 Valid Architectures **的内容对应。

问题三
---
再次编译提示错误：**lb： library not found for -lPods 或者 lb： library not found for -libPods-AFNetworking.a** 

解决方法：

* 编辑 **Podfile**
* 在上面添加的内容中再加上 **config.build_settings['ONLY_ACTIVE_ARCH'] = 'NO'**

```
post_install do |installer_representation|
    installer_representation.project.targets.each do |target|
        target.build_configurations.each do |config|
            config.build_settings['ARCHS'] = 'armv7 armv7s'
            config.build_settings['ONLY_ACTIVE_ARCH'] = 'NO'
        end
    end
end  
```
* 这是因为 64 位下项目中的 **Build Active Architecture Only** 默认是 NO，而 CocoaPods 中的是 YES。

问题四
---
通过 cocospods 引入 BaiduMapAPI ，执行 pod update 终端提示如下：

```
[!] The `YYW [Debug]` target overrides the `FRAMEWORK_SEARCH_PATHS` build setting defined in `Pods/Target Support Files/Pods/Pods.debug.xcconfig'. This can lead to problems with the CocoaPods installation
    - Use the `$(inherited)` flag, or
    - Remove the build settings from the target.

[!] The `YYW [Release]` target overrides the `FRAMEWORK_SEARCH_PATHS` build setting defined in `Pods/Target Support Files/Pods/Pods.release.xcconfig'. This can lead to problems with the CocoaPods installation
    - Use the `$(inherited)` flag, or
    - Remove the build settings from the target.
```
解决办法： 

* 选择程序 **xxx -> TARGETS -> Build Settings** 
+ 找到 **Library Search Path** 添加 **$(inherited)**
* 编译运行通过