实现一个叶子widget。



一个LeafWidget是一个widget，没有槽，对于儿子。

LeafWidgets是通常地目的作为构建块，对于聚合widgets。



SLeafWidget从SWidget继承，构造函数设置SWidget的bCanHaveChildren为false。



```c++
//设置可见性
virtual void SetVisibility(TAttribute<EVisibility> InVisibility) override final;
```



```c++
//开始SWigdte重载

/*
	从SWidget重载
	
	LeafWidgets提供它们的一个可见的表达。它们这样做，通过添加DrawElements到OutDrawElements。
	DrawElements应当让它们的位置设置为绝对坐标在窗口空间，对于这个目的，Slate系统提供了
	AllottedGeometry参数。AllottedGeometry描述了分配的空间对于这个widget的可见性。
	
	只要可能，LeafWidgets应当避免解决布局属性，看Textblock对于一个例子
*/
virtual in32 OnPaint(const FPaintArgs& Args, const FGeometry& AllottedGeometry, const FSlateRect& MyCullingRect, FSlateWindowElementList& OutDrawElements, int32 LayerId, const FWidgetStyle& InWidgetStyle, bool bParentEnabled ) const override = 0;
	
/*
从SWidget中重载

LeafWidgets应当计算它们DesiredSize仅基于其视觉表现。这里没有需要去考虑子部件，因为LeafWidgets在定义上没有。例如，TextBlock widget简单地测量了需要的区域，去显示它的文本，对于给定的字体和字体大小。
*/
virtual FVector2D ComputeDesiredSize(float) const override = 0;

/*
从SWidget中重载。

叶子widgets永远不会有儿子。
*/
virtual FChildren* GetChildren() override;//返回NoChildrenInstance

virtual void OnArrangeChildren( const FGeometry& AllottedGeometry, FArrangedChildren& ArrangedChildren ) const override;//里面是空的，没有任何东西要去安排

//FNoChildren的共享实例，对于所有没有儿子的widgets
static FNoChildren NoChildrenInstance;
```



```c++
void SLeafWidget::SetVisibility(TAttribute<EVisibility> InVisibility)
{
	SWidget::SetVisibility(InVisibility);
}
```































