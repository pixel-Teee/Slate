ArrangeChildren的结果是总是返回作为一个FArrangedChildren。

FArrangedChildren支持一个过滤器，是非常有用的，用于排除那些不想要的可见性的widgets。



```c++
//数据成员

EVisibility VisibilityFilter;//可见性过滤器

//注意，FArrangedWidget是带FGeometry的widget
typedef TArray<FArrangedWidget, TInlineAllocator<4>> FArrangedWidgetArray;//安排好的widget数组


```

