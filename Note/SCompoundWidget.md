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





