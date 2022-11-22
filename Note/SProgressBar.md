进度条



```c++
//SProgressBar Fill Type
UENUM(BlueprintType)
namespace EProgressBarFillType
{
	enum Type//类型
    {
        LeftToRight,
        RightToLeft,
        FillFromCenter,
        FillFromCenterHorizontal,
        FillFromCenterVertical,
        TopToBottom,
        BottomToTop,
    };
}

//SProgress Fill Style
UEnum(BlueprintType)
namespace EProgressBarFillStyle
{
    enum Type
    {
        //一个遮罩被用于绘制填充的图像
        Mask,
        //填充的图像被缩放到填充的百分比
        Scale
    }
}

/** 一个process bar widget*/
class SLATE_API SProgressBar : public SLeafWidget
    
/** process bar的style */
const FProgressBarStyle* Style;

/** 显示在process bar上的文本 */
TAttribute<TOptional<float>> Percent;

EProgressBarFillType::Type BarFillType;

EProgressBarFilleStyle::Type BarFillType;

/** 背景图片用于progress bar*/
const FSlateBrush* FillImage;

/** 前景图片用于progress bar*/
const FSlateBrush* FillImage;

/** 用于marquee mode的图片*/
const FSlateBrush* MarqueeImage;

/** 填充颜色和透明度*/
TAttribute<FVector2D> FillColorAndOpacity;

/** 边界padding */
TAttribute<FVector2D> BorderPadding;

/** 值去驱动进度条动画 */
float MarqueeOffset;//marquee(跑马灯)offset

/** 引用到widgets，现在激活的timer*/
TWeakPtr<FActiveTimerHandle> ActiveTimerHandle;

/** widget现在被tick的概率，当slate的sleep的模式被激活的时候*/
float CurrentTickRate;

/** widget可以tick的最慢的速度，当处于slate睡眠模式的时候*/
float MinimumTickRate;

//有一大堆设置的函数

//几个比较重要的函数
void Construct(const FArguments& InArgs);

virtual int32 OnPaint( const FPaintArgs& Args, const FGeometry& AllottedGeometry, const FSlateRect& MyCullingRect, FSlateWindowElementList& OutDrawElements, int32 LayerId, const FWidgetStyle& InWidgetStyle, bool bParentEnabled ) const override;

virtual FVector2D ComputeDesiredSize(float) const override;

virtual bool ComputeVolatility() const override;//计算易变性(Volatility)

//有几个私有成员
/** 控制widget被tick的速度，当处于slate的睡眠模式下的时候 */
void SetActiveTimerTickRate(float TickRate);

//更新跑马灯Active Timer
void UpdateMarqueeActiveTimer();
```



```c++
//Construct

void SProgressBar::Construct(const FArguments& InArgs)
{
	//construct这里除了设置属性，还调用了父类的SetCanTick
	SetCanTick(false);
	
	//设置跑马灯ActiveTimer
	UpdateMarqueeActiveTimer();
}
```



























