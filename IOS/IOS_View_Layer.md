
## Basic

* 视图是 UIView 对象， 或者 UIView 子类对象。
* 视图知道如何绘制自己。
* 视图可以处理事件。
* 视图会按照层次结构排列，位于视图层次结构顶端的是应用窗口。

## 视图层次及结构

任何一个应用都有且只有一个 UIWindow 对象。它像一个容器，包含应用中的所有视图。应用需要在启动时，创建并设置 UIWindow 对象，然后为其添加视图。

> 树状模型

绘制过程分为两步：

* 层次结构中的每个视图分别绘制自己。视图会将自己会知道图层上，每个 UIView 对象都有一个 layer 属性，指向 CALayer 类的对象。可以将图层看成是一个位图图像。

* 所有视图的图层组合合成一副图像，绘制到屏幕上。

## Core Graphics

绘制核心在此框架中，有部分 OC 类整合了部分方法，若无法用 OC 对象处理，则可通过 CG 方法直接处理。
  
# 视图生命周期方法

* application:didFinishLaunchingWithOptions 在该方法中设置和初始化应用窗口的根视图控制器。该方法只会在应用启动完毕后调用一次，之后如果从其他应用切回本应用，则该方法不会再次调用。如果关闭应用后台进程并重新启动应用，该方法才再次被调用。

* initWithNibName:bundle 该方法是 UIViewController 的指定初始化方法，创建视图控制器时，就会调用该方法。注意：某些情况下，需要在同一个应用中创建多个相同的 UIViewController 子类对象，每次创建一个该类的对象时，都会调用一次该类的 initWithNibName:bundle: 方法。

* loadView 可以覆盖该方法，使用代码设置视图控制器的 view 属性。

* viewDidLoad 可以覆盖该方法，设置使用 NIB 文件创建的视图对象。该方法会在视图控制器加载完视图后被调用。

* viewWillAppear 可以覆盖该方法，设置使用 NIB 文件创建的视图对象。该方法和 viewDidAppear:会在每次视图控制器的 view 显示在屏幕上时被调用；相反，viewWillDisappear:和viewDidDisappear:方法会在每次视图控制器的 view 从屏幕上消失时被调用。