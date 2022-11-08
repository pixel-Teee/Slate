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





























