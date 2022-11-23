# SCompoundWidget

CompoundWidget是构建大多数非primitive widgets的基础。



CompoundWidget有一个protected成员，**叫做ChildSlot。**



```c++
//返回大小缩放系数对于这个widget
//返回大小缩放系数
const FVector2D GetContentScale() const;

//设置内容的缩放，对于这个widget
void SetContentScale(TAttribute<FVector2D> InContentScale);

//获取widget的颜色
FLinearColor GetColorAndOpacity() const;

//设置widget的颜色
//参数 InColorAndOpacity ColorAndOpacity被应用到这个widget还有所有它的内容
void SetColorAndOpacity(TAttribute<FLinearColor> InColorAndOpacity);//ColorAbdOpacityAttribute

//设置widget的前景色
//参数 InColor 要去设置的颜色
void SetForegroundColor(TAttribute<FSlateColor> InForegroundColor);//ForegroundColorAttribute

//数据成员
/** 应用到这个widget的内容的布局缩放，有用的对于动画*/
TSlateAttribute<FVector2D> ContentScaleAttribute;

/**应用到这个widget的颜色和透明度还有它所有的儿子*/
TSlateAttribute<FLinearColor> ColorAndOpacityAttribute;

/**可选的前景色，将从所有这个widget的内容中继承*/
TSlateAttribute<FSlateColor> ForegroundColorAttribute;

/**包含这个widget的子代的slot*/
FCompoundWidgetOneChildSlot ChildSlot;

//有三个函数，返回TSlateAttributeRef类型，根据SWidget和TSlateAttribute进行构造
TSlateAttributeRef<FSlateColor> GetForegroundColorAttribute() const 
{ return TSlateAttributeRef<FSlateColor>{SharedThis(this), ForegroundColorAttribute};}
```



descendants n.后代；子孙

```c++
struct FCompoundWidgetOneChildSlot : ::TSingleWidgetChildrenWithBasicLayoutSlot<EInvalidWidgetReason::None>
{
	friend SCompoundWidget;
	using ::TSingleWidgetChildrenWithBasicLayoutSlot<EInvalidateWidgetReason::None>::TSingleWidgetChildrenWithBasicLayoutSlot;
};//一个非常重要的结构体，复合widget一个儿子slot，从TSingleWidgetChildrenWithBasicLayoutSlot继承

//单一WidgetChildren和基础的布局槽
```



```c++
//绘制函数
int32 SCompoundWidget::OnPaint(const FPaintArgs& Args,
const FGeometry& AllottedGeometry,
const FSlateRect& MyCullingRect,
FSlateWidowElementList& OutDrawElements,
int32 LayerId,
const FWidgetStyle& InWidgetStyle,
bool bParentEnabled)
{
    //一个compound widget只绘制它的儿子
    FArrangedChildren ArrangedChildren(EVisibility::Visible);
    {
        this->ArrangeChildren(AllottedGeometry, ArrangedChildren);
    }
    
    //这里可能有零个元素在数组，如果我们的儿子折叠/隐藏了
    if(ArrangedChildren.Num() > 0)
    {
        const bool bShouldBeEnabled = ShouldBeEnabled(bParentEnabled);
        
        check(ArrangedChildren.Num() == 1);
        FArrangedWidget& TheChild = ArrangedChildren[0];
        
        FWidgetStyle CompoundedWidgetStyle = FWidgetStyle(InWidgetStyle)
        .BlendColorAndOpacityTint(GetColorAndOpacity())
        .SetForegroundColor(bShouldBeEnabled ? GetForegoundColor() : GetDisabledForegroundColor());
        
        int32 Layer = 0;
		Layer = TheChild.Widget->Paint(Args.WithNewParent(this), TheChild.Geometry, MyCullingRect,
                OutDrawElements, LayerId + 1, CompoundedWidgetStyle, bShouldBeEnabled);//这里LayerId + 1了
        return Layer;
    }
    return LayerId;
}
```



```c++
FChildren* SCompoundWidget::GetChhildren()
{
	return &ChildSlot;
}
```



```c++
FVector2D SCompundWidget::ComputeDesiredSize(float) const
{
	EVisibility ChildVisibility = ChildSlot.GetWidget()->GetVisibility();//获取可见性
	if(ChildVisibility != EVisibility::Collapsed)//不是折叠，计算大小
	{
		return ChildSlot.GetWidget()->GetDesiredSize() + ChildSlot.GetPadding().GetDesiredSize();//加上了padding大小
	}
	return FVector2D::ZeroVector;
}
```



```c++
void SCompoundWidget::OnArrangeChildren(const FGeometry& AllottedGeometry, FArrangedChildren& ArrangedChildren) const
{
	ArrangedSingleChild(GSlateFlowDirection, AllottedGeometry, ArrangedChildren, ChildSlot, GetContentScale());
}
```



```c++
SCompoundWidget::SCompoundWidget()
:ChildSlot(this),
ContentScaleAttribute(*this, FVector2D(1.0f, 1.0f)),
ColorAndOpacityAttribute(*this, FLinearColor::White),
ForegroundColorAttribute(*this, FSlateColor::UseForeground)
{

}
```



```c++
void SCompoundWidget::SetVisibility(TAttribute<EVisibility> InVisibility)
{
	SWidget::SetVisibility(MoveTemp(InVisibility));
}
```



































