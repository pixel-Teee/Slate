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

const uint8 bHasRenderTransform : 1;//这个或许可以用来判断是否有父亲
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
	const FSlateRenderTransform& InLocalRenderTransform,
    const FVector2D& InLocalRenderTransformPivot,
    const FSlateLayoutRenderTransform& ParentAccumulatedLayoutTransform,
    const FSlateRenderTransform& ParentAccumulatedRenderTransform
) : Size(InLocalSize)
    , Scale(1.0f)
    , AbsolutePosition(0.0f, 0.0f)
    , AccumulatedRenderTransform(
		Concatenate(
        	//转换pivot到局部空间，并且使得它为原点
            Inverse(TransformPoint(FScale2D(InLocalSize), InLocalRenderTransformPivot)),
            //应用render transform在局部空间围绕pivot的中心
            InLocalRenderTransform,
            //转换pivot point回去
            TransformPoint(FScale2D(InLocalSize), InLocalRenderTransformPivot),
            //应用layout transform next
            InLocalLayoutTransform,
            //最终地应用parent accumulated transform，输入我们到根
            ParentAccumulatedRenderTransform
        ),
    bHasRenderTransform(true)
	)
{
    FSlateLayoutTransform AccumulatedLayoutTransform = Concatenate(InLocalLayoutTransform, ParentAccumulatedLayoutTransform);
    //hack去允许使得FGeometry的不可变公共成员去捕获滥用
    const_cast<FVector2f&>( AbsolutePosition ) = FVector2f(AccumulatedLayoutTransform.GetTranslation());
    const_cast<float&>( Scale ) = AccumulatedLayoutTransform.GetScale();
    const_cast<FVector2f&>( Position ) = FVector2f(InLocalLayoutTransform.GetTranslation());
}
//从这个函数可以看出一个，布局transform是不受轴点的影响的，而render transform则会受到影响

/*
制作一个新的geometry，本质上是层次的根(没有父亲transform去继承)
对于一个根Widget，LayoutTransform是通常的window DPI scale + window offset

LocalSize 几何的大小，在局部空间
LayoutTransform 几何的Layout Transform
返回 新的根geometry
*/
static FGeometry MakeRoot(const FVector2D& InLocalSize, const FSlateLayoutTransform& LayoutTransform)
{
    return FGeometry(InLocalSize, LayoutTransform, FSlateLayoutTransform(), FSlateRenderTransform(), false);
}

/*
创建一个子geometry，相对于这个，使用一个给定的局部空间大小，layout transform，还有render transform
例如，一个widget有5 * 5 margin，将创建一个geometry，对于它的儿子内容，有一个Translate(5, 5)的LayoutTransform，
并且一个LocalSize 10 单位，小于它所持有的

LocalSize 在局部空间的儿子geometry
LayoutTransform 相对于这个Geometry的新的儿子的Layout transform，从儿子的layout空间到这个widget的layout space
RenderTransform 新儿子的只渲染的transform，被应用，在layout transform之前，对于只有渲染的目的
RenderTransformPivot 在Render transform归一化局部空间的Pivot

返回 新的儿子geometry
*/
FGeometry MakeChild(const FVector2D& InLocalSize, const FSlateLayoutTransform& LayoutTransform, const FSlateRenderTransform& RenderTransform, const FVector2D& RenderTransformPivot) const
{
		return FGeometry(InLocalSize, LayoutTransform, RenderTransform, RenderTransformPivot, GetAccumulatedLayoutTransform(), GetAccumulatedRenderTransform());
}

//还有几个MakeChild

/*
创建一个儿子 geometry+widget，相对于这个，使用给定的LayoutGeometry

ChildWidget 这个geometry，对其创建的child widget
LayoutGeometry child的Layout geometry

返回新的child geometry
*/
/*
FLayoutGeometry就两个成员，FSlateLayoutToTransform LocalToParent
FVector2f LocalSize
*/
FArrangedWidget MakeChild(const TSharedRef<SWidget>& ChildWidget, const FLayoutGeometry& LayoutGeometry) const;
```









































