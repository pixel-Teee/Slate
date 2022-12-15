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























