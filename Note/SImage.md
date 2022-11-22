实现一个widget，显示一个image，使用一个渴望的宽度和高度。



数据成员：

```c++
//用于绘制image的slate brush，我们可以无效
TSlateAttribute<const FSlateBrush*> ImageAttributes;

//颜色和透明度的缩放，对于这个image
TSlateAttribute<FSlateColor> ColorAndOpacityAttribute;

//当描述，忽略内容渴望的大小，并且警告the.HeightOverride作为Box的渴望高度
TSlateAttribute<TOptional<FVector2D>> DesiredSizeOverrideAttribute;

//翻转image，如果流方向的定位(localization)是从右到左
bool bFlipForRightToLeftFlowDirection;
```



```c++
//这些函数基本就是给属性赋值
/*
构造这个widget
参数，InArgs，声明数据对于这个widget
*/
void Construct(const FArguments& InArgs);

/** 设置ColorAndOpacity属性*/
void SetColorAndOpacity(TAttribute<FSlateColor> InColorAndOpacity);

/** 设置ColorAndOpacity属性*/
void SetColorAndOpacity(FLinearColor InColorAndOpacity);

/** 设置Image属性*/
void SetImage(TAttribute<const FSlateBrush*> InImage);

/** 设置SizeOverride属性*/
void SetDesiredSizeOverride(TAttribute<TOptional<FVector2D>> InDesiredSizeOverride);
```



ComputeDesiredSize会根据DesiredSizeOverrideAttribute是否被设置，从而返回DesiredSizeOverrideAttribute的大小，还是ImageBrush的大小。



```c++
//OnPaint函数

int32 SImage::OnPaint(const FPaintArgs& Args, const FGeometry& AllottedGeometry, const FSlateRect& MyCullingRect, FSlateWindowElementList& OutDrawElements, int32 LayerId, const FWidgetStyle& InWidgetStyle, bool bParentEnabled) const
{
	const FSlateBrush* ImageBrush = ImageAttribute.Get();//获取FSlateBrush
    
    if((ImageBrush != nullptr) && (ImageBrush->DrawAs != ESlateBrushDrawType::NoDrawType))
    {
        const bool bIsEnabled = ShouldBeEnabled(bParentEnabled);//是否应当开启
        
        const ESlateDrawEffect DrawEffects = bIsEnabled ? ESlateDrawEffect::None : ESlateDrawEffect::DisabledEffect;
        //开启了，ESlateDrawEffect为ESlateDrawEffect::None，否则为ESlateDrawEffect::DisabledEffect
        const FLinearColor FinalColorAndOpacity(InWidgetStyle.GetColorAndOpacityTint() * ColorAndOpacityAttribute.Get().GetColor(InWidgetStyle) * ImageBrush->GetTint(InWidgetStyle));//style color/opacity image做了混合
        
        if(bFlipForRightToLeftFlowDirection && GSlateFlowDirection == EFlowDirection::RightToLeft)
        {
            //翻转
            const FGeometry FlippedGeometry = AllottedGeometry.MakeChild(FSlateRenderTransform(FScale2D(-1, 1)));
            FSlateDrawElement::MakeBox(OutDrawElements, LayerId, FlippedGeometry.ToPaintGeometry(), ImageBrush, DrawEffects, FinalColorAndOpacity);//制作一个Box
        }
        else
        {
            FSlateDrawElement::MakeBox(OutDrawElements, LayerId, AllottedGeometry.ToPaintGeometry(), ImageBrush, DrawEffects, FinalColorAndOpacity);//制作一个Box
        }
    }
    
    return LayerId;//SImage不处理传进来的LayerId，直接返回
}
```



注意，这个函数使用了一个函数参数中的变量，**bParentEnabled**，来判断要不要绘制。



































































