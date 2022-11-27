```c++
//widget可能的不同类型的无效
enum class EInvadliateWidgetReason : uint8
{
	None = 0,
	
	//使用布局无效化，如果你的widget需要去改变渴望的大小，这是一个昂贵的无效化，那么不要去使用，如果你所需要的只是重新绘制一个widget
	Layout = 1 << 0,
	
	//使用，当widget的绘画已经更改，但是不影响大小的时候
	Paint = 1 << 1,
	
	//使用，如果只是widget的不稳定性被调整
	Volatility = 1 << 2,
	
	//一个child被添加或者被移除，这个意味着prepass(预先准备)和布局
	ChildOrder = 1 << 3,
	
	//一个widget渲染的transform被改变
	RenderTransform = 1 << 4,
	
	//改变可见性(这个暗示着布局)
	Visibility = 1 << 5,
	
	//属性被绑定或者没有被绑定(它被SlateAttributeMetaData所使用)
	AttributeRegistration = 1 << 6,
	
	//重新cache所有这个widget的儿子的渴望大小递归地(这个暗示着布局)
	Prepass = 1 << 7,
	
	/*
	使用Paint无效化，如果你修改了一个普通的属性，涉及到绘制或者缩放。
	额外地，如果更改的属性无论如何都会影响不稳定性，那是重要的，你无效化不稳定性，那么它可以被重新计算
	并且cache
	*/
    PaintAndVolatility = Paint | Volatility,
    
    /*
    使用Layout无效化，如果你修改了一个普通的属性，涉及到绘制或者缩放。
	额外地，如果更改的属性无论如何都会影响不稳定性，那是重要的，你无效化不稳定性，那么它可以被重新计算
	并且cache
    */
    LayoutAndVolatility = Layout | Volatility,
};

```

