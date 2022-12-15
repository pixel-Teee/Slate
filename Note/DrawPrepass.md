绘制预处理过程。



注释指出，这个DrawPrepass函数，这是一个Pre-pass阶段，在绘制窗口到计算几何大小以及reshape自动缩放的窗口的时候。



```c++
void FSlateApplication::DrawPrepass(TSharedPtr<SWindow> DrawOnlyThisWindow)
//FSlateApplication::PrivateDrawWindows去调用这个函数，传入一个nullptr
{
	TSharedPtr<SWindow> ActiveModalWindow = GetActiveModalWindow();//nullptr
    if(ActiveModalWindow.IsValid()){...}//针对模态窗口
    else if(DrawOnlyThisWindow.IsValid()){...}//只针对传入的参数的窗口
    else
    {
        //绘制所有窗口
        for(const TSharedRef<SWindow>& CurrentWindow : SlateWindows)
        {
            PrepassWindowAndChildren(CurrentWindow);//调用PrepassWindowAndChildren
        }
    }
}
```



DrawPrepass针对不同类型的窗口进行不同的处理。



SlateWindows是被这个应用所持有的**所有顶层窗口**，它们在这里被追踪在一个**平台无关**的方式。



之后DrawPrepass针对每个顶层窗口，调用一个静态函数，PrepassWindowAndChildren，传入顶层窗口。



```c++
static void PrepassWindowAndChildren(TSharedRef<SWindow> WindowToPrepass)
{
	//参数：需要预处理的Window
	
    //如果是运行在DS服务器上的，直接退出
    if(IsRunningDedicatedServer())
    {
        return;
    }
    
    //窗口可见的条件，可见外加不是最小化的
    const bool bIsWindowVisible = WindowToPrepass->IsVisible() && !WindowToPrepass->IsWindowMinimized();
    
    //如果窗口是可见的，或者该窗口的任何子代窗口都需要预处理的话
    //DoAnyWindowDescendantsNeedPrepass针对这个窗口的子代，判断它们是否为可见的，并且不是最小化的，是一个递归的过程
    //其实，这里有一些些小小的疑问，竟然父窗口看不见的话，为啥子代要进行判断呢？
    if(bIsWindowVisible || DoAnyWindowDescendantsNeedPrepass(WindowToPrepass))
    {
        WindowToPrepass->ProcessWindowInvalidation();//处理窗口的无效
        WindowToPrepass->SlatePrepass(FSlateApplication::Get().GetApplicationScale() *
        WindowToPrepass->GetNativeWindow()->GetDPIScaleFactor());//slate prepass，参数这里传入了一个布局缩放系数，还有一个DPI缩放因子
        
        if(bIsWindowVisible && WindowToPrepass->IsAutoSized())
        {
            //调用Resize函数
            WindowToPrepass->Resize(WindowToPrepass->GetDesizedSizeDesktopPixels());
        }
        
        //获取该窗口的儿子窗口
        TArray<TSharedRef<SWindow>, TMemStackAllocator<>> ChildWindows(WindowToPrepass->GetChildWindows());
        for(const TSharedRef<SWindow>& ChildWindow : ChildWindows)
        {
            PrepassWindowAndChildren(ChildWindow);//递归调用
        }
    }
}
```



PrepassWindowAndChildren是一个递归的过程，首先调用一些预处理函数，对当前这个父窗口进行计算，

然后再便利每个子窗口，**调用每个子窗口的PrepassWindowAndChildren。**



# ProcessWindowInvalidation

```c++
void SWindow::ProcessWindowInvalidation()//处理窗口无效
{
	//本身允许快速更新并且全局的无效化开启了
	if(bAllowFastUpdate && GSlateEnableGlobalInvalidation)
	{
		ProcessInvalidation();
	}
}
```



```c++
//SWindow是继承自FSlateInvalidationRoot和SCompoundWidget的
bool FSlateInvalidationRoot::ProcessInvalidation()
{
	//Widgets是否需要重新绘制
	bool bWidgetsNeedRepaint = false;
    
    if(!bNeedsSlowPath){...}
    
    if(!bNeedsSlowPath){...}
    
    if(!bNeedsSlowPath){...}
    
    //重新运行任何ChildOrder无效化，属性可能已经添加了新的ChildOrder
    if(!bNeedsSlowPath && WidgetsNeedingPreUpdate->Num() > 0){...}
    
    if(!bNeedsSlowPath){...}
    
    if(!bNeedsSlowPath){...}
    
    if(bNeessSlowPath){...}
    
 	return bWidgetsNeedRepaint   
}
```



有6个针对bNeedsSlowPath的处理块。



以下每一个都是针对不同语句块的内容：

```c++
ProcessPreUpdate();//处理预更新

ProcessAttributeUpdate();//处理属性更新

ProcessPreUpdate();//处理预更新，这里调用了两次，不太明白

//放置所有的widgets在VolatileUpdate list在WidgetsNeedingPostUpdate
//需要后处理更新的widgets
WidgetsNeedingPrepassUpdate->Heapify();//堆化?
WidgetsNeedingPostUpdate->Heapify();//堆化?
//堆化是否指的是类似二叉堆那样的东西，把一个数组变成堆?
//这里for循环，创建widget volatile 更新迭代器
//是FSlateInvalidationWidgetList的迭代器
for(FSlateInvalidationWidgetList::FWidgetVolatileUpdateIterator Iterator = 
FastWidgetPathList->CreateWidgetVolatileUpdateIterator(true); Iterator.IsValid(); Iterator.Advance())
{
	FSlateInvalidationWidgetList::InvalidationWidgetType& InvalidationWidget = 
	(*FastWidgetPathList)[Iterator.GetCurrentIndex()];//获取无效的widget，从FastWidgetPathList里获取
	
	//是否VolatilePrepass
	if(InvalidationWidget.bIsVolatilePrepass)
	{
		//无效理由
		InvalidationWidget.CurrentInvalidationReason |= EInvalidationWidgetReason::Layout;
		WidgetsNeedingPrepassUpdate->HeapPushUnique(InvalidationWidget);//WidgetsNeedingPrepassUpdate放入InvalidationWidget
	}
}

ProcessPrepassUpdate();//注意这个和前面的区别，多了一个prepass

FinalUpdateList.Reset(WidgetsNeedingPostUpdate->Num());
bWidgetsNeedRepaint = ProcessPostUpdate();

//这个针对bNeedsSlowPath
WidgetsNeedingPreUpdate->Reset(true);
WidgetsNeedingPrepassUpdate->Reset(true);
WidgetsNeedingPostUpdate->Reset(true);
FinalUpdateList.Reset();
CachedElementData->Empty();
bWidgetsNeedRepaint = true;
```



PreUpdate、PrepassUpdate、PostUpdate，3个过程。



















