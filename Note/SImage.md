实现一个widget，显示一个image，使用一个渴望的宽度和高度。



数据成员：

```c++
//用于绘制image的slate brush，我们可以无效
FInvalidatableBrushAttribute Image;

//颜色和透明度的缩放，对于这个image
TAttribute<FSlateColor> ColorAndOpacity;

//翻转image，如果流方向的定位(localization)是从右到左
bool bFlipForRightToLeftFlowDirection;

//被调用，当鼠标按压在image上的时候
FPointerEventHandler OnMouseButtonDownHandler;
```



```c++
/*
构造这个widget
参数，InArgs，声明数据对于这个widget
*/
void Construct(const FArguments& InArgs);
```





























