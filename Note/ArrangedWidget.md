# FArrangedWidget

一对：Widget还有它的Geometry。Widgets填充一个列表的WidgetGeometries，当它们安排他们的儿子的时候。

查看SWidget::ArrangeChildren。



```c++
//数据成员
/** widget的几何 */
FGeometry Geometry;

/** 被安排的widget */
TSharedRef<SWidget> Widget;
```



# FWidgetAndPointer

FWidgetAndPointer继承自FArrangedWidget



具体用法不明？



```c++
#if WITH_EDITOR
TSharedRef<const FVirtualPointerPosition> PointerPosition;//指向位置
#endif

TOptional<FVirtualPointerPosition> OptionalPointerPosition;//虚拟指向位置
```



























