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
//这里相当于开始绘制了
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
                WidgetPtr->SlatePrepass(WidgetPtr->GetPrepassLayoutScaleMultiplier());
            }
        }
    }
    
}
```

























