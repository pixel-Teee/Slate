抽象基类，对于Slate widgets。



停止。不要直接地从WIDGET继承！



继承：

Widget不是意味着直接地被继承。相反考虑从LeafWidget或者Panel继承，表示预期的用例(?)并且提供**一组简洁的方法去重写。**



SWidget是对于所有交互的Slate实体的基类。SWidget的公共接口描述了所有事物，一个Widget可以做到的，因此相当复杂。



事件：

事件在Slate被实现作为虚函数，Slate系统将调用Widget，目的是去通知Widget关于一个重要事件(occurrence)(例如，一个按键)或者询问Widget，依据一些信息(例如，什么鼠标光标应当被显示)。



Widget提供一个默认实现，对于大多数事件；默认实现不做什么事情并且不处理事件。



一些事件能够回复系统通过返回一个FReply，FCursorReply，或者相似的对象。



![image-20221107230909766](Image/SWidget/image-20221107230909766.png)

SWidget继承FSlateControlledConstruction，TSharedFromThis<SWidget>，还有大量的友元。



2个Construct函数，从4.27开始，SWidget::Construct应当不被直接地调用。使用SNew或者SAssignNew去创建一个SWidget。



# 通用的事件

Paint，被调用去告诉一个widget去绘制它自己(还有它的儿子)。

widget应当响应，通过使用FDrawElements来填充OutDrawElements，来表示它还有任何它的儿子。



## Paint

```c++
int32 Paint(const FPaintArgs& Args, const FGeometry& AllottedGeometry, const FSlateRect& MyCullingRect, FSlateWindowElementList& OutDrawElements, int32 LayerId, const FWidgetStyle& InWidgetStyle, bool bParentEnabled) const;
```



Args，所有必要的参数去绘制这个widget(todo umg：移动所有的参数到这个结构体)。

AllottedGeometry，FGeometry描述了一个区域widget应当出现的。

MyCullingRect，分配的clipping rectangle(裁剪矩形)对于这个widget还有它的儿子。

OutDrawElements，一个列表的FDrawElements去填充输出。

LayerId，这个widget应当被渲染到的层。

InColorAndOpacity，颜色还有透明度去被应用到所有正在绘制的小部件的所有后代。

bParentEnabled，true，如果这个widget的父亲被开启。

**返回最大的layer ID被获取，通过这个widget或者任何它的儿子。**

## Tick

使用Geometry Tick这个widget。重写在派生类，但是总是调用父亲的实现。

AllocatedGeometry，分配的控件对于这个widget。

InCurrentTime，现在的绝对实时时间。

InDeltaTime，实时的时间，自从上一次Tick。

```c++
virtual void Tick(const FGeometry& AllottedGeometry, const double InCurrentTime, const float InDeltaTime);
```



## Key Input

被调用，当焦点被给予给这个widget。这个事件不会冒泡(bubble)？

MyGeometry，widget接受事件的几何。

InFocusEvent，FocusEvent。

返回，**是否事件被处理**，伴随着其它可能的行为。



```c++
virtual FReply OnFocusReceived(const FGeometry& MyGeometry, const FFocusEvent& InFocusEvent);
```



---





被调用，当这个widget丢失焦点。这个事件没有冒泡(bubble)？

InFocusEvent，FocusEvent。

```c++
virtual void OnFocusLost(const FFocusEvent& InFocusEvent);
```



---





每当在新旧焦点路径中的所有小部件上焦点路径发生变化时调用。

```c++
	UE_DEPRECATED(4.13, "Please use the newer version of OnFocusChanging that takes a FocusEvent")
	virtual void OnFocusChanging(const FWeakWidgetPath& PreviousFocusPath, const FWidgetPath& NewWidgetPath);
```

使用新版本的OnFocusChaing，获取一个FocusEvent。

---



```c++
virtual void OnFocusChanging(const FWeakWidgetPath& PreviousFocusPath, const FWidgetPath& NewWidgetPath, const FFocusEvent& InFocusEvent);
```

新版本，多了一个FFocusEvent。

---



被调用，在一个字符进入，当这个widget有键盘焦点。

MyGeometry，接受事件的widget的几何。

InCharacterEvent，字符事件。

返回是否事件被处理，伴随着其它可能的行为。

```c++
virtual FReply OnKeyChar(const FGeometry& MyGeometry, const FCharacterEvent& InCharacterEvent);
```

---



被调用，在一个键被按下，当这个widget或者这个widget的一个儿子有焦点的时候，

如果一个widget处理了这个事件，OnKeyDown将不会传递到焦点的widget。

这个事件主要是为了去允许父widgets去消耗一个事件在一个子widget处理它，并且它应当被使用，当这里没有更好的可选的事件。

MyGeometry，接受事件的widget的几何。

InKeyEvent，Key Event。

返回是否事件被处理，伴随着其它可能的行为。

```c++
virtual FReply OnPreviewKeyDown(const FGeometry& MyGeometry, const FKeyEvent& InKeyEvent);
```

---



被调用，在一个key被传递，当这个widget有焦点的时候(这个事件冒泡，如果没有被处理)。

MyGeometry，接受事件的widget的几何。

InKeyEvent，Key Event。

返回是否事件被处理，伴随着其它可能的行为。

```c++
virtual FReply OnKeyDown(const FGeometry& MyGeometry, const FKeyEvent& InKeyEvent);
```

---



被调用，在一个键被释放的时候，当这个widget有焦点的时候。

MyGeometry，接受事件的widget的几何。

InKeyEvent，Key Event。

返回是否事件被处理，伴随着其它可能的行为。

```c++
virtual FReply OnKeyUp(const FGeometry& MyGeometry, const FKeyEvent& InKeyEvent);
```



当支持**模拟(anaglog)**的按钮上的模拟值发生变化时调用。

MyGeometry，接受事件的widget的几何。

InKeyEvent，Key Event。

返回是否事件被处理，伴随着其它可能的行为。

```c++
virtual FReply OnAnalogValueChanged(const FGeometry& MyGeometry, const FAnalogInputEvent& InAnalogInputEvent);
```



大概有个AnalogValue，表示模拟值？**不大明确，FAnalogInputEvent。**



## Mouse Input

系统调用这个方法去通知widget，一个鼠标按钮被按下，在它立马。这个事件是冒泡的。

MyGeometry，接受事件的widget的几何。

MouseEvent，关于输入事件的信息。

返回是否事件与系统采取行动的可能请求一起被处理。

```c++
virtual FReply OnMouseButtonDown(const FGeometry& MyGeometry, const FPointerEvent& MouseEvent);
```

---



就像OnMouseButtonDown，但是tunnels代替冒泡。

如果这个事件被处理，OnMouseButtonDown将不会被发送。

谨慎使用这个此事件，因为预览事件通常会使得UI更难推理。

```c++
virtual FReply OnPreviewMouseButtonDown(const FGeometry& MyGeometry, const FPointerEvent& MouseEvent);
```

---



系统调用这个方法去通知widget，一个鼠标按钮被释放在它立马。这个事件是冒泡的。

MyGeometry，接受事件的widget的几何。

MouseEvent，关于输入事件的信息。

返回是否事件与系统采取行动的可能请求一起被处理。

```c++
virtual FReply OnMouseButtonUp(const FGeometry& MyGeometry, const FPointerEvent& MouseEvent);
```

---



系统调用这个方法去通知widget，一个鼠标移动到它这里。这个事件是冒泡的。

MyGeometry，接受事件的widget的几何。

MouseEvent，关于输入事件的信息。

返回是否事件与系统采取行动的可能请求一起被处理。

```c++
virtual FReply OnMouseMove(const FGeometry& MyGeometry, const FPointerEvent& MouseEvent);
```



系统将使用这个事件去通知一个widget，一个光标进入了它。事件是使用了一个自定义的冒泡策略。

MyGeometry，接受事件的widget的几何。

MouseEvent，关于输入事件的信息。

```c++
virtual void OnMouseEnter(const FGeometry& MyGeometry, const FPointerEvent& MouseEvent);
```



---

系统将使用这个事件去通知一个widget，光标已经离开了它。事件是使用了一个自定义冒泡策略。

MouseEvent，关于输入事件的信息。



```c++
virtual void OnMouseLeave(const FPointerEvent& MouseEvent);
```



---

被调用，当鼠标滚轮旋转的时候。这个事件已经冒泡。

MouseEvent，关于输入事件的信息。

返回是否事件被处理，跟随其它可能的行为。

```c++
virtual FReply OnMouseWheel(const FGeometry& MyGeometry, const FPointerEvent& MouseEvent);
```



---

系统要求鼠标下的每个小部件提供一个光标。这个事件是冒泡的。

FCusorReply::Unhandled()，如果事件没有被处理，否则返回FCusorReply::Cursor()。

```c++
virtual FCursorReply OnCursorQuery(const FGeometry& MyGeometry, const FPointerEvent& CursorEvent) const;
```



也就是鼠标下的widget要提供光标？



---

OnCursorQuery指定光标类型后，系统会要求鼠标下的每个小widget去映射该光标到小widget。

这个事件是冒泡的。

TOptional<TSharedRef<SWidget>>()，如果你没有一个映射，否则返回一个Widget去展示。

```c++
virtual TOptional<TSharedRef<SWidget>> OnMapCursor(const FCursorReply& CursorReply) const;
```

---

被调用，当一个鼠标按钮被双击的时候。**重写这个在派生类。**

InMyGeometry Widget几何。

MouseEvent，关于输入事件的信息。

返回是否事件被处理，跟随其它可能的行为。

```c++
virtual FReply OnMouseButtonDoubleClick(const FGeometry& InMyGeometry, const FPointerEvent& InMouseEvent);
```

---

被调用，当Slate想去可视化tooltip。

如果没有人处理这个事件，Slate将使用默认的tooltip可视化。

**如果你重写这个事件，你应当可能地返回true。**

TooltipContent 我可能想去可视化的TooltipContent。

返回true，如果这个widget可视化了tooltip内容，例如，事件被处理了。

```c++
virtual bool OnVisualizeTooltip(const TSharedPtr<SWidget>& TooltipContent);
```



---

可视化一个新的pop-up，如果可能的话。如果这个小部件不能承载弹出内容，你将返回一个指向该层的无效指针。

返回的FPopupLayer允许你去移除pop-up，当你已经完成了它。



PopupContent，要尝试并托管的小部件覆盖在小部件的顶部。

返回一个FPopupLayer，如果这个widget支持持有它。你可以调用Remove()在这个去消耗pop-up。

```c++
virtual TSharedPtr<FPopupLayer> OnVisualizePopup(const TSharedRef<SWidget>& PopupContent);
```

---

被调用，当Slate检测一个widget开始被拖(dragged)。

用法：

一个widget可以请求Slate去检测一个drag。

OnMouseDown()答复，使用FReply::Handled().DetectDrag(SharedThis(this))

Slate将么发送一个OnDragDetected()事件或者什么都不做。

如果用户释放了一个鼠标按钮或者离开widget在一个拖动(drag)被触发(可能用户开始在一个非常的边缘)，那么没有事件将被发送。



InMyGeometry，Widget几何。

InMouseEvent，触发drag的MouseMove。

```c++
virtual FReply OnDragDetected(const FGeometry& MyGeometry, const FPointerEvent& MouseEvent);
```

---

## Drag And Drop(DragDrop)



被调用，在drag和drop，当drop(落下)进入一个widget。



进入/离开 事件在slate意味着作为轻量的通知。

那么我们不想去捕获鼠标或者设置焦点作为对这些的响应。

然而，OnDragEnter必须也支持外部的APIs(例如，OLE Drag/Drop)

这些要求我们让它们知道是否我们可以处理内容，在被抓取OnDragEnter。



妥协是去返回一个can_handled/cannot_handle布尔值，而不是一个完整的FReply。



MyGeometry，接收事件的widget的几何。

DragDropEvent，drag和drop事件。

返回一个reply，表示是否DragDropEvent的内容可以潜在地被处理通过这个widget。

```c++
virtual void OnDragEnter(const FGeometry& MyGeometry, const FDragDropEvent& DragDropEvent);
```

---

被调用在drag和drop，当drag离开一个widget。

DragDropEvent，drag和drop事件。

```c++
virtual void OnDragLeave(const FDragDropEvent& DragDropEvent);
```



被调用在drag和drop的事件，当鼠标被抓取拖过一个widget。

MyGeometry，接受事件的widget的几何。

DragDropEvent，drag和drop事件。

返回一个回复，表示是否这个事件被处理。

```c++
virtual FReply OnDragOver(const FGeometry& MyGeometry, const FDragDropEvent& DragDropEvent);
```



---

被调用，当用户放下什么东西在一个widget，终止drag和drop。

MyGeometry，接受事件的widget的几何。

DragDropEvent，drag和drop事件。

返回一个回复，表示是否这个事件被处理。

```c++
virtual FReply OnDrop(const FGeometry& MyGeometry, const FDragDropEvent& DragDropEvent);
```



## TOUCH and GESTURES

触摸还有手势



被调用，当用户执行一个手势在一个trackpad(触控板)。这个事件是冒泡的。

GestureEvent，手势事件。

返回是否一个事件是否冒泡，伴随着可能的行为。

```c++
virtual FReply OnTouchGesture(const FGeometry& MyGeometry, const FPointerEvent& GestureEvent);
```

---

被调用，当一个touchpad(触摸板)开始的时候(手指向下)，就是可以用笔在屏幕上画画的。

InTouchEvent，触摸事件生成的。

```c++
virtual FReply OnTouchStarted(const FGeometry& MyGeometry, const FPointerEvent& InTouchEvent);
```

---

被调用当一个触摸板触摸被移开的时候(鼠标移开的时候)。

InTouchEvent，生成的触摸事件。

```c++
virtual FReply OnTouchMoved(const FGeometry& MyGeometry, const FPointerEvent& InTouchEvent);
```

---

被调用，当一个触摸板触摸被终止(手指抬起(lifted))。

InTouchEvent，生成的触摸事件。

```c++
virtual FReply OnTouchEnded(const FGeometry& MyGeometry, const FPointerEvent& InTouchEvent);
```

---

被调用，当一个触摸板强迫改变的时候。

InTouchEvent，生成的触摸事件。

```c++
virtual FReply OnTouchForceChanged(const FGeometry& MyGeometry, const FPointerEvent& TouchEvent);
```

---

被调用，当一个触摸板，在TouchStarted移动后。

InTouchEvent，生成的触摸事件。

```c++
virtual FReply OnTouchFirstMove(const FGeometry& MyGeometry, const FPointerEvent& TouchEvent);
```

---

被调用，当motion(动作)被检测的时候(控制器或设备)

例如，有人倾斜或者摇晃控制器。

InMotionEvent，生成的motion事件。

```c++
virtual FReply OnMotionDetected(const FGeometry& MyGeometry, const FMotionEvent& InMotionEvent);
```

---

被调用，检测是否我们应当渲染焦点画刷(focus brush)。

InFocusCause，焦点的结果。

```c++
virtual TOptional<bool> OnQueryShowFocus(const EFocusCause InFocusCause) const;
```

---

Popups可以显示在一个新的OS WINDOW或者通过一个OVERLAY在一个存在的window。

这个可以被显示地设置在SMenuAnchor，或者可以被决定，通过一个scoping widget。

一个scoping widget可以回复OnQueryPopupMethod()去驱动所有它的子代poup方法。



全屏的游戏不能够传唤一个新窗口，那么游戏SViewports将回复EPopupMethod::UserCurrentWindow。

这个使得所有在它们的menu anchors使用现在的窗口。

```c++
virtual FPopupMethodReply OnQueryPopupMethod() const;
```

---



一个不知道有什么用的函数

```C++
virtual TSharedPtr<FVirtualPointerPosition> TranslateMouseCoordinateForCustomHitTestChild(const TSharedRef<SWidget>& ChildWidget, const FGeometry& MyGeometry, const FVector2D& ScreenSpaceMouseCoordinate, const FVector2D& LastScreenSpaceMouseCoordinate) const;
```

转换鼠标坐标对于自定义的碰撞检测儿子。

---

所有的指针(鼠标，触摸，触笔(stylvs)，等)事件已经从该帧路由(routed)。

这是一个widget的机会**去执行任何累积的数据。**

```C++
virtual void OnFinishedPointerInput();
```

---

所有的键(键盘，gamepay，joystick，等等)输入从这帧已经被路由。

这是一个widget的机会**去执行任何累积的数据。**

```c++
virtual void OnFinishedKeyInput();
```

---

被调用，当鼠标移过widget的window，去决定是否我们应当报告是否OS确切的特性应当被激活在这个位置(例如一个title bar grip，系统菜单，等等)。

通常，你不需要去重载这个函数。

---

返回，光标所在的窗口区域，或者EWindowZone::Unspecified，如果没有特殊的行为被需要。

```c++
virtual EWindowZone::Type GetWindowZoneOverride() const;
```

---

accessibility n.可访问性

```c++
virtual TSharedRef<class FSlateAccessibleWidget> CreateAccessibleWidget();
```

---





































