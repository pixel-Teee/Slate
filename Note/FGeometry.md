表示位置，大小，还有一个widget的绝对位置在Slate种。

一个geometry的绝对位置是通常地屏幕坐标或者窗口坐标，依赖于geometry初始的位置。

Geometries是通常地和一个SWidget指针伴随，依据是否提供关于一个确切widget的消息(查看FArrangedWidget)。

一个几何的父亲是通常地被认为相应parent widget的Geometry。



之前的FVector2D LocalSize、float AbsoluteScale、FVector2f AbsolutePosition、FVector2f LocalPosition替换成了

**FSlateRenderTransform AccumulatedRenderTransform。**

```c++
/*
连接的Render Transform。实际的transform被用于渲染。
形式作为：Concat(LocalRenderTransform, LocalLayoutTransform, Parent->AccumulatedRenderTransform)

对于渲染，绝对坐标将总是在窗口控件(相对于根窗口)
*/
FSlateRenderTransform AccumulatedRenderTransform;

const uint8 bHasRenderTransform : 1;
```



```c++
/*
构造一个新的geometry，和一个给定的大小在局部控件，被依附于一个父geometry，和给定的布局还有render transform

InLocalSize geometry的大小在局部空间
InLocalLayoutTransform 来自于local space的布局transform到父geometry的局部空间
InLocalRenderTransform 一个只渲染用的transform在local space，将被追加到LocalLayoutTransform，当渲染的时候
InLocalRenderTransformPivot 在归一化局部空间的pivot，对于局部渲染transform
ParentAccumulatedLayoutTransform parent widget的累加的layout transform，AccumulatedLayoutTransform = Concat(LocalLayoutTransform, ParentAccumulatedLayoutTransform)
ParentAccumulatedRenderTransform parent widget的累加的render transform，AccumulatedRenderTransform = Concat(LocalRenderTransform, LocalLayoutTransform, ParentAccumulatedRenderTransform)
*/

FGeometry(
	const FVector2D& InLocalSize,
	const FSlateLayoutTransform& InLocalLayoutTransform,
	
)
```









































