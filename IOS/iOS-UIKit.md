# iOS UIKit 框架

## UI 视图结构
视图为 N 叉树状数据结构，视图可拥有 N 个子视图和 1 个父视图。由此视图具有 N 叉树的特点，在 Responder Chain，绘制层中有着重要作用。

## Responder Chain

关键点：

1. UIReponder 是所有视图的超类，使所有视图具有响应相关特性。

### hit-test view
iOS 系统检测到手指触摸操作会将其打包成一个 UIEvent 对象，并放入当前 Application 的事件队列。单例 UIApplication 从事件队列中取得事件对象，交给单例 UIWindow 处理。

UIWindow 首先会调用其内容视图的 `func hitTest(_ point: CGPoint, with event: UIEvent) -> UIView?` ，内容视图会调用`func point(inside point: CGPoint, with event: UIEvent?) -> Bool`，确认是否在自身范围内，然后根据情况递归调用子视图的 `hitTest`方法。

结束条件：

1. 不在自身范围内，hitTest 返回 nil。结束递归
2. 在自身范围内，并没有子视图，hitTest 返回自身

最终找到触发的事件的叶子视图。从 root 节点到目标节点的路径组成了一个**Responder Chain**。

*注*

1. hitTest 方法将会忽略隐藏(hidden=YES)的视图，禁止用户操作(userInteractionEnabled=YES)的视图，以及alpha级别小于0.01(alpha<0.01)的视图。
2. 如果一个子视图的区域超过父视图的bound区域(父视图的clipsToBounds 属性为NO，这样超过父视图bound区域的子视图内容也会显示)，那么正常情况下对子视图在父视图之外区域的触摸操作不会被识别,因为父视图的pointInside:withEvent:方法会返回NO,这样就不会继续向下遍历子视图了。
3. 我们可以重写hitTest:withEvent:来达到某些特定的目的

> 这个过程称作 **hit-test view**。

### 事件传递
事件传递是按照之上形成的 **Reponder Chain** 依次向上传递。

事件上传过程中，若一直没有被处理，最终会被丢弃。处理后也可调用 super 方法，将事件往上传递。

## View

[TO READ](http://www.cocoachina.com/ios/20150828/13244.html)