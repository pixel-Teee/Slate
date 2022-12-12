一个Paint geometry包含窗口空间(绘制空间)的信息去绘制一个元素到屏幕上。

它包含元素的大小在元素的局部空间，以及必要的transform去定位元素在窗口空间。



DrawPosition/DrawSize/DrawScale被维护由于遗留原因，但是被遗弃的：

DrawPosition和DrawSize是已经定位的并且缩放的对于被绘制代码所立即消耗。



DrawScale只被应用到绘制图元的内部视角，例如，线宽，3x3网格边距(margins)。



```c++
/*
	已遗弃。绘制应当只发生在局部空间去保证render transform工作。
	警告：遗弃的成员表示是布局空间，并且不会统计进render transforms。
	
	我们将绘制的渲染空间的位置。
*/
FVector2f DrawPosition;

/*
	已遗弃。绘制应当只发生在局部空间，去保证render transform工作。
	警告：遗弃的成员表示是布局空间，并且不会统计进render transforms。
	
	只影响独立绘制元素的绘制视角，例如，线宽，3x3网格边距。
*/
float DrawScale;

/*获取在局部空间的geometry的大小。必须调用CommitTransformIfUsingLegacyConstructor()，如果遗弃的构造函数被使用*/
const FVector2D& GetLocalSize() const { return LocalSize; }

/*访问最终的render transform。必读调用CommitTransformIfUsingLegacyConstructor()，如果遗弃的构造函数被使用*/
const FSlateRenderTransform& GetAccumulatedRenderTransform() const { return AccumulatedRenderTransform; }

/*
支持可变的成员被构造在窗口空间，并且可能在之后可变，因为所有遗弃的成员是公共的。

在这些情况，我们延迟RenderTransform和LocalSize的调用，直到渲染时间，保证所有的成员改变已经完成。
警告：遗弃的用法，不支持render transforms！
*/

//提交transform，如果使用遗弃的构造函数
void CommitTransformIfUsingLegacyConstructor() const
{
    //bUsingLegacyConstructor为false，则直接返回，即不适应遗弃的构造函数
    if(!bUsingLegacyConstructor) return;
    
    AccumulatedRenderTransform = FSlateRenderTransform(DrawScale, FVector2D(DrawPosition));
    
    FSlateLayoutTransform AccumulatedLayoutTransform = FSlateLayoutTransform(DrawScale, FVector2D(DrawPosition));
    
    LocalSize = TransformVector(Inverse(AccumulatedLayoutTransform), FVector2D(DrawSize));
}

bool HasRenderTransform() const { return bHasRenderTransform; }

private:
//可变的去支持遗弃的构造函数，不统计进render transform
mutable FVector2f DrawSize;

//可变的去支持遗弃的构造函数
mutable Fvector2D LocalSize;

//最终的render transform对于绘制。从局部空间变换到窗口空间对于绘制元素。可变的去支持遗弃的构造函数。
mutable FSlateRenderTransform AccumulatedRenderTransform;

//支持遗弃的构造函数
uint8 bUsingLegacyConstructor : 1;

uint8 bHasRenderTransform : 1;

//默认构造
FPaintGeometry()
    :DrawPosition(0.0f, 0.0f)
    ,DrawScale(1.0f)
    ,DrawSize(0.0f, 0.0f)
    ,LocalSize(0.0f, 0.0f)
    ,AccumulatedRenderTransform()
    ,bUsingLegacyConstructor(true)
    ,bHasRenderTransform(true)
    {
        
    }

/*
创建和初始化一个新的实例
InLocalSize 在局部空间的大小
InAccumulatedLayoutTransform 累加的layout transform(对于一个FGeometry)
InAccumulatedRenderTransform 累计的render transform(对于一个FGeometry)
*/
FPaintGeometry( const FSlateLayoutTransform& InAccumulatedLayoutTransform, const FSlateRenderTransform& InAccumulatedRenderTransform, const FVector2D& InLocalSize, bool bInHasRenderTransform)
		: DrawPosition(InAccumulatedLayoutTransform.GetTranslation())
		, DrawScale(InAccumulatedLayoutTransform.GetScale())
		, DrawSize(0.0f, 0.0f)
		, LocalSize(InLocalSize)
		, AccumulatedRenderTransform(InAccumulatedRenderTransform)
		, bUsingLegacyConstructor(false)
		, bHasRenderTransform(bInHasRenderTransform)
	{ 
	}
```



如果FPaintGeometry是默认构造的，那么都是一些默认参数，CommitTransformIfUsingLegacyConstructor会执行，后续还可以修改FPaintGeometry。

如果提供了一些参数，则调用CommitTransformIfUsingLegacyConstructor，直接返回，相当于不可变的FPaintGeometry。





