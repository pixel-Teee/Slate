# FPaintArgs

SWidget::OnPaint和SWidget::Paint使用FPaintArgs作为它们其一的参数，目的是为了减轻传递多个字段的负担。

```c++
//数据成员
/**最根部的grid，只有窗口应当设置这个，并且只有无效化panels应当修改它 */
FHittestGrid& RootGrid;

/**现在碰撞的测试grid，当存在嵌套的无效化面板的时候，可能存在多个网格，这是widgets应当总是去添加的*/
FHittestGrid& CurrentGrid;

FVector2D WindowOffset;

const SWidget* PaintParentPtr;

double CurrentTime;
float DeltaTime;
uint8 bInheritedHittestability : 1;
uint8 bDeferredPainting : 1;
```



```c++
FPaintArgs(const SWidget* PaintParent, FHittestGrid& InRootHittestGrid, FHittestGrid& InCurrentHitTestGrid, FVector2D InWindowOffset, double InCurrentTime, float InDeltaTime);

FPaintArgs(const SWidget* PaintParent, FHittestGrid& InRootHittestGrid, FVector2D InWindowOffset,
double InCurrentTime, float InDeltaTime);//相比于上面，少了一个InCurrentHitTestGrid

//插入自定义的碰撞测试路径
FPaintArgs InsertCustomHitTestPath(const SWidget* Widget, TSharedRef<ICustomHitTestPath> CustomHitTestPath) const;

//参数：继承的碰撞的能力
void SetInheritedHittestability(bool InInheritedHittestability)
{
    bInheritedHittestability = InInheritedHittestability;
}

bool GetInheritedHittestability() const
{
    return bInheritedHittestability;//继承的碰撞能力
}

FHittestGrid& GetHittestGrid() const
{
    return CurrentGrid;
}

const SWidget* GetPaintParent() const
{
    return PaintParentPtr;
}

FVector2D GetWindowToDesktopTransform() const
{
    return WindowOffset;
}

double GetCurrentTime() const
{
    return CurrentTime;
}

double GetDeltaTime() const
{
    return DeltaTime;
}

void SetDeferredPaint(bool InDeferredPaint)
{
    bDeferredPainting = InDeferredPaint;
}

bool GetDeferredPaint() const
{
    return bDeferredPainting;
}
```



```c++
FPaintArgs FPaintArgs::InsertCustomHitTestPath(const SWidget* Widget, TSharedRef<ICustomHitTestPath> CustomHitTestPath) const
{
    //这里就类型转换了下，然后调用了InsertCustomHitTestPath
	const_cast<FHittestGrid&>(CurrentGrid).InsertCustomHitTestPath(Widget, CustomHitTestPath);
	return *this;
}
```



































