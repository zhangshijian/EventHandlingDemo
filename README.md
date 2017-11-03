
# iOS事件处理，响应者和响应链

## 概述

应用程序使用响应者对象来接收和处理事件，属于`UIResponder`类的实例对象都是响应者对象，常见的子类包括`UIView`、`UIViewController`和`UIApplication`。响应者接收到原始事件数据后，必须处理该事件或者将此事件转发给另一个响应者。当应用程序接收到一个事件时，UIKit会自动将该事件指向最合适的响应者对象，此响应者对象称为第一响应者。应用程序中不存在单一的响应者链，UIkit定义了如何将对象从一个响应者传递到下一个响应者的默认规则，但是我们可以随时通过覆盖响应者对象中的相应属性来更改这些规则。下图显示了应用程序中的默认响应者链，其界面包含一个label，一个text field，一个button和两个background view。如果text field没有处理某个事件，UIKit会将事件发送到text field的父视图对象，如果事件还是未被处理，UIKit会继续发送该事件到此视图的父视图，直到发送到window的根视图，然后响应者链从根视图转移到持有此根视图的视图控制器，再从视图控制器转移到window。如果window不处理这个事件，UIKit会将事件传递给`UIApplication`对象，如果该对象是`UIResponder`类的实例并且当前响应链不包含该对象，那么UiKit可能将该事件传递给应用程序的委托对象。

![图1-1](https://docs-assets.developer.apple.com/published/7c21d852b9/f17df5bc-d80b-4e17-81cf-4277b1e0f6e4.png)

## 确定事件的第一响应者

对于每种类型的事件，UIKit都会指定一个第一响应者，并首先将事件发送到该对象，第一响应者根据事件的类型而有所不同：

- 触摸事件(Touch events)：第一响应者是触摸所在屏幕位置的视图。
- 按压事件(Press events)：第一响应者是按压焦点所在的响应者。
- 摇晃事件(Shake-motion events)：第一响应者是由我们自己或者UIKit指定为第一响应者的对象。
- 远程控制事件(Remote-control events)：第一响应者是由我们自己或者UIKit指定为第一响应者的对象。
- 编辑菜单消息(Editing menu messages)：第一响应者是由我们自己或者UIKit指定为第一响应者的对象。

> **注意**：与加速计、陀螺仪和磁力计相关的运动事件不遵循响应者链，Core Motion会将这些事件直接传递给我们指定的对象。有关更多信息，可以参看[Core Motion Framework](https://developer.apple.com/library/content/documentation/Miscellaneous/Conceptual/iPhoneOSTechOverview/CoreServicesLayer/CoreServicesLayer.html#//apple_ref/doc/uid/TP40007898-CH10-SW27)。

控件使用动作消息直接与其关联的目标对象进行通信。当用户与控件交互时，控件会调用其`target`对象的`action`方法——换句话说，控件会向目标对象发送一个动作消息。动作消息不属于事件，但是它也可以使用响应者链。当控件的`target`对象为`nil`时，UIKit会从`target`对象开始顺着响应链寻找，直到找到实现了对应`action`方法的对象。例如，编辑菜单就使用这种方式去搜索实现了方法名为`cut:`、`copy:`、`paste:`的对象。

如果一个视图具有附加的手势识别器，手势识别器会先于视图接收到触摸和按压事件。如果所有视图的手势识别器都无法识别它们的手势，那么事件就会被传递给视图进行处理。如果视图没有处理它们，UIKit会将事件传递给响应者链。有关使用手势识别器处理事件的更多信息，可以参看[Handling UIKit Gestures](https://developer.apple.com/documentation/uikit/touches_presses_and_gestures/handling_uikit_gestures)。

## 确定哪个响应者包含触摸事件

UIKit使用基于视图的命中测试来确定触摸事件发生的位置。具体来说，UIKit将触摸位置与视图层次结构中的视图对象的边界进行比较。`UIView`的`hitTest:withEvent:`方法会遍历视图层次结构，寻找包含指定触摸事件的最深的子视图，这个视图就会成为触摸事件的第一响应者。

> **注意**：如果触摸位置在视图的边界之外，`hitTest:withEvent:`方法会忽略此视图以及其所有子视图。因此，当视图的`clipsToBounds`的属性为`NO`时，即使子视图恰好包含该触摸事件，但子视图超出了视图的边界，这个子视图也会被忽略。

UIKit会将每个触摸事件永久指定给包含它的视图，当触摸开始时，UIKit会为每个触摸手势创建一个`UITouch`对象，直到触摸结束之后才会释放`UITouch`对象。随着触摸位置或其他参数的改变，UIKit会使用新信息更新`UITouch`对象，唯一不变的属性是触摸手势所包含的视图。即使触摸位置移动到触摸手势包含的原始视图之外，触摸手势所包含的视图也不会改变。

## 更改响应者链

可以通过覆写响应者对象的`nextResponder`属性来更改响应者链，许多UIKit类已经覆盖此属性并返回了特定的对象。

- `UIView`对象：如果这个视图是视图控制器的根视图，那么下一个响应者就是这个视图控制器；否则，下一个响应者就是它的父视图。
- `UIViewController`对象：如果视图控制器的视图是window的根视图，则下一个响应者就是window；如果视图控制器是被另一个视图控制器呈现的，则下一个响应者是这个呈现视图控制器。
- `UIWindow`对象：window的下一个响应者是`UIApplication`对象。
- `UIApplication`对象：当应用程序的委托对象是`UIResponder`类的实例，而不是视图、视图控制器或者应用程序对象本身时，其下一个响应者就是应用程序的委托对象。


## hit-testing

`hitTest:withEvent:`方法会遍历当前视图层，并调用每个子视图的`pointInside:withEvent:`方法来确定哪个子视图来接收触摸事件。如果`pointInside:withEvent:`方法返回`YES`，则会同样遍历子视图的视图层，直到找到包含指定点的最上层视图。如果一个视图不包含该点，那么此视图层分支会被忽略掉。可以通过覆写`hitTest:withEvent:`方法来隐藏子视图中的触摸事件。

此方法会忽略掉被隐藏，已禁用用户交互或者`alpha`小于**0.01**的视图对象。在确定命中时，此方法不会考虑视图的内容。因此，即使指定点位于该视图内容的透明部分，仍然可以返回这个视图。
