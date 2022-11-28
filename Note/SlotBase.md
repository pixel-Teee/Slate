# FSlotBase

```c++
//slot是一个SWidget的容器，可以被FChildren使用
class FSlotBase;

//数据成员
//持有slot的children
const FChildren* Owner;

//slot的内容widget
TSharedRef<SWidget> Widget;

//有个空的内置的结构体
struct FSlotArguments{};//槽参数

//访问持有slot的FChildren，owner可以无效，当slot没有被依附的时候
const FChildren* GetOwner() const { return Owner; }

//访问持有slot的widget，owner可以无效，当slot没有被依附的时候
SWidget* GetOwnerWidget() const
{
    return GetOwner() ? &(GetOwner()->GetOwner()) : nullptr;
}

/**
设置slot的owner
Slots不能赋值给不同的父亲
*/
void SetOwner(const FChildren& Children)
{
 	if(Owner != &InChildren)
    {
        Onwer = &InChildren;
        AfterContentOrOwnerAssigned();//在内容或者owner被赋值后
    }
}

//依附slot目前持有的child widget
void AttachWidget(const TSharedRef<SWidget>& InWidget)
{
    TSharedRef<SWidget> LocalWidget = Widget;
    DetachParentFromContent();//分离父亲从内容中
    Widget = InWidget;
    AfterContentOrOwnerAssigned();//在内容或者owner被赋值后
}

/**
访问在现在的槽的widget
这里将总是一个widget在slot，有时它是SNullWidget实例
*/
const TSharedRef<SWidget>& GetWidget() const
{
	return Widget;
}

/**
移除widget，从它现在的槽
移除的widget被返回，那么操作可以在它上面执行
如果null widget被存储，一个无效化的shared ptr被代替返回
*/
const TSharedRef<SWidget> DetachWidget();

//无效化widget的持有者
void Invalidate(EInvalidateWidgetReason InvalidateReason);
```

