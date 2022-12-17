Widget代理



```c++
FSlateInvalidationWidgetIndex Index;

FSlateInvalidationWidgetIndex ParentIndex;

FSlateInvalidationWidgetIndex LeafMostChildIndex;//索引

EInvalidateWidgetReason CurrentInvalidateReason;//理由

FSlateInvalidationWidgetVisiblity Visiblity;//可见性

union
{
	struct
    {
    public:
        //widget在pre update list里面?
        uint8 bContainedByWidgetPreHeap : 1;
        //widget在post update list里面?
        uint8 bContainedByWidgetPostHeap : 1;
        //widget在pending prepass list里面?
        uint8 bContainedByWidgetPrepassList : 1;
        //widget是否是一个Invalidation Root，Cache SWidget::Advanced_IsInvalidationRoot的值
        uint8 bIsInvalidationRoot : 1;
        //widget是否有volatile prepass flag
        uint8 bIsVolatilePrepass : 1;
    };
    uint8 PrivateFlags;
};

//持有一个widget的裸指针
SWidget* Widget;

//构造函数
FWidgetProxy::FWidgetProxy(SWidget& InWidget)
: Widget(&InWidget)
, Index(FSlateInvalidationWidgetIndex::Invalid)
, ParentIndex(FSlateInvalidationWidgetIndex::Invalid)
, LeafMostChildIndex(FSlateInvalidationWidgetIndex::Invalid)
, CurrentInvalidateReason(EInvalidationWidgetReason::None)
, Visibility()
, PrivateFlags(0)
{
    
}
```



```c++
//这个Update是DrawWindowAndChildren一路调用过来的，也就是计算完缓存和shape后，一路调用过来
FWidgetProxy::FUpdateResult FWidgetProxy::Update(const FPaintArgs& PaintArgs, FSlateWindowElementList& OutDrawElements)
{
	TSharedRef<Swidget> CurrentWidget = GetWidgetAsShared();
	
	//如果外部的layer id保留索引1，这里没有改变
	FUpdateResult Result;
	//有任何更新标识符，就调用Repaint
	if(CurrentWidget->HasAnyUpdateFlags(EWidgetUpdateFlags::NeedsRepaint | EWidgetUpdateFlags::NeedsVolatilePaint))
	{
		Result = Repaint(PaintArgs, OutDrawElements);
	}
    else
    {
        EWidgetUpdateFlags PreviousUpdateFlag = CurrentWidget->UpdateFlags;
        //需要活动时间更新
        if(CurrentWidget->HasAnyUpdateFlags(EWidgetUpdateFlags::NeedsActiveTimerUpdate))
        {
            CurrentWidget->ExecuteActiveTimers(PaintArgs.GetCurrentTime(), PaintArgs.GetDeltaTime());
        }
        
        if(CurrentWidget->HasAnyYpdateFlags(EWidgetUpdateFlags::NeedsTick))
        {
            //获取持久状态
            const FSlateWidgetPersistentState& MyState = CurrentWidget->GetPersistentState();
            
            //调用Tick，会调用子类的Tick虚函数
            CurrentWidget->Tick(MyState.DesktopGeometry, PaintArgs.GetCurrentTime(), PaintArgs.GetDeltaTime());
        }
    }
    
    return result;
}
```



```c++
//FWidgetProxy的Repaint函数
FWidgetProxy::FUpdateResult FWidgetProxy::Repaint(const FPaintArgs& PaintArgs, FSlateWindowElement& OutDrawElements) const
{
	SWidget* WidgetPtr = GetWidget();
    //获取持久状态
    const FSlateWidgetPersistentState& MyState = WidgetPtr->GetPersistentState();
    
    //获取裁剪索引
    const int32 StartingClipIndex = OutDrawElements.GetClippingIndex();
    
    //获取裁剪管理器到正确的状态
    const bool bNeedsNewClipState = MyState.InitialClipState.IsSet();
    if(bNeedsNewClipState)
    {
        //填充裁剪状态
        OutDrawElements.GetClippingManager().PushClippingState(MyState.InitialClipState.GetValue());
    }
    
    //之前的用户索引
    const int32 PrevUserIndex = PaintArgs.GetHittestGrid().GetUserIndex();
    
    PaintArgs.GetHittestGrid().SetUserIndex(MyState.IncomingUserIndex);
    GSlateFlowDirection = MyState.IncomingFlowDirection;
    
    FPaintArgs UpdatedArgs = PaintArgs.WithNewParent(MyState.PaintParent.Pin().Get());
    UpdatedArgs.SetInheritedHittestability(MyState.bInheritedHittestability);
    UpdatedArgs.SetDeferredPaint(MyState.bDeferredPainting);
    
    int32 PrevLayerId = MyState.OutgoingLayerId;
    
    //如果开启了全局的无效化
    if(GSlateEnabledGlobalInvalidation)
    {
        if(WidgetPtr->HasAnyUpdateFlags(EWidgetUpdateFlags::NeedsVolatilePaint))
        {
            if(WidgetPtr->SholdInvalidatePrepassDueToVolatility())
            {
                WidgetPtr->MarkPrepassAsDirty();
                //注意，这个slate pre pass函数，比较重要，计算自己的DesiredSize并缓存
                WidgetPtr->SlatePrepass(WidgetPtr->GetPrepassLayoutScaleMultiplier());
            }
        }
    }
    
}
```



总结：也就是说一个类的不同的函数在不同的阶段被进行调用，一个函数用于计算，一个函数用于绘制。



```c++
void SWidget::SlatePrepass()
{
	SlatePrepass(FSlateApplication::Get().GetApplicationScale());
}
```



```c++
void SWidget::SlatePrepass(float InLayoutScaleMultipiler)//布局缩放乘积
{
    //如果不是在全局快速更新路径或者需要pre pass
	if(!GSlateIsOnFastUpdatePath || bNeedsPrepass)
    {
        //如果注册了属性
        if(HasRegisteredSlateAttribute() && IsAttributesUpdatesEnabled() && !GSlateIsOnFastProcessInvalidation)
        {
            FSlateAttributeMetaData::UpdateAllAttributes(*this, FSlateAttributeMetaData::EInvalidationPermission::AllowInvalidationIfConstructed);
        }
        
        Prepass_Internal(InLayoutScaleMultiplier);
    }
}
```



```c++
void SWidget::Prepass_Internal(float InLayoutScaleMultiplier)
{
	PrepassLayoutScaleMultiplier = InLayoutScaleMultiplier;
	
	bool bShouldPrepassChildren = true;
	
	if(bHasCustonPrepass)
	{
		bShouldPrepassChildren = CustomPrepass(InLayoutScaleMultiplier);
	}
    
    if(bCanHaveChildren && bShouldPrepassChildren)
    {
        //cache child desized sizes first
        //cache儿子的desized大小第一步
        //widget的desized size是一个它的儿子们的大小的函数
        FChildren* MyChildren = this->GetChildren();
        const int32 NumChildren = MyChildren->Num();
        Prepass_ChildLoop(InLayoutScaleMultipler, MyChildren);      
    }
    
    {
        //cache这个widget的desized大小
        CacheDesizedSize(PrepassLayoutScaleMultiplier.Get(1.0f));
        bNeesPrepass = false;
    }
}
```



这里出现了3个比较关键的函数，CustomPrepass，Prepass_ChildLoop，CacheDesizedSize。



这个CustomPrepass判断widget是否有自定义的Prepass。



```
void SWidget::Prepass_ChildLoop(float InLayoutScaleMultiplier, FChildren* MyChildren)
{
	int32 ChildIndex = 0;
	MyChildren->ForEachWidget([this, &ChildIndex, InLayoutScaleMultiplier](SWidget& Child))
	{
		//儿子有注册属性
		const bool bUpdateAttributes = Child.HasRegisteredSlateAttribute() && 
		Child.IsAttributesUpdatesEnabled() && !GSlateIsOnFastProcessInvalidation;
		
		//更新属性
		if(bUpdateAttributes)
		{
			FSlateAttributeMetaData::UpdateOnlyVisiblityAttributes(Child, FSlateAttributeMetaData::EInvalidationPermission::AllowInvalidationIfConstructed);
		}
		
		if(Child.GetVisibility != EVisibility::Collapsed)
		{
			if(bUpdateAttributes)
			{
				FSlateAttributeMetaData::UpdateExceptVisibilityAttributes(Child, FSlateAttributeMetaData::EInvalidationPermission::AllowInvalidationIfConstructed);
			}
			
			const float ChildLayoutScaleMultiplier = bHasRelativeLayoutScale
			? InLayoutScaleMultipler * GetRelativeLayoutScale(ChildIndex, InLayoutScaleMultiplier) : InLayoutScaleMultiplier;
			
			//递归
			//recur : descend down the widget tree
			Child.Prepass_Internal(ChildLayoutScaleMultiplier);
		}
		else
		{
			//如果儿子widget被折叠，我们需要去存储它将有的新的布局缩放，当它最终地可见并且
			//无效化它的prepass，那么它可以获取，当它的可见性最终地无效化
			Child.MarkPrepassAsDirty();
			Child.PrepassLayoutScaleMultiplier = bHasRelativeLayoutScale
			? InLayoutScaleMultiplier * GetRelativeLayoutScale(ChildIndex, InLayoutScaleMultiplier) : InLayoutScaleMuliplier;
		}
		++ChildIndex;
	});
}
```



```c++
//CacheDesizedSize

void SWidget::CacheDesizedSize(float InLayoutScaleMultiplier)
{
	//cache 这个widget的需要的大小
	SetDesizedSize(ComputeDesiredSize(InLayoutScaleMultiplier));
}
```



```c++
/*
计算必要的理想的大小，去显示这个widget。对于聚合widgets(例如：panels)这个大小应当包含
必要的大小去展示所有它的儿子，CacheDesizedSize()保证它的子代的大小被计算并且被cached在parents前面，
那么它是安全地去调用GetDesizedSize()对于任何children，当实现这个方法的时候

注意，ComputeDesizedSize()意味着作为一个助手，对于开发者。
它不意味着非常roubust在一些情况。
如果你的widget是模拟一个bouncing ball，你可以只要返回一个理由的大小，例如：160 * 160，让程序员去设置一个有理由的规则，缩放bouncy ball模拟。
*/
FVector2D ComputeDesizedSize(float LayoutScaleMultiplier) const
{
	//参数的LayoutScaleMultiplier几乎可以忽略，只对文本测量有影响
	//返回需要的大小
	//SButton的这个函数，直接返回了Image的大小
}
```





















