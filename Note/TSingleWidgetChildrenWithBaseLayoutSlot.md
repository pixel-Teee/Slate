**TSingle Widget Children With Basic Layout Slot**



单个widget children，和基础的布局槽。



```c++
//一个FChildren，只有一个child，并且支持alignment和padding
template<EInvalidationWidgetReason InPaddingInvalidationReason = 
EInvalidateWidgetReason::Layout>

class TSingleWidgetChildrenWithBasicLayoutSlot :
public TSingleWidgetChildrenWithSlot<TSingleWidgetChildrenWithBasicLayoutSlot
<InPaddingInvalidationReason>>,
public TPaddingSingleWidgetSlotMixin<TSingleWidgetChildrenWithBasicLayoutSlot
<InPaddingInvalidationRease>, InPaddingInvalidationReason>,
public TAlignmentSingleWidgetSlotMixin<TSingleWidgetChildrenWithBasicLayoutSlot
<InPaddingInvalidationRease>>
{
private:
    using ParentType = TSingleWidgetChildrenWithSlot<TSingleWidgetChildrenWithBasicLayoutSlot<InPaddingInvalidationReason>>;
    
    using PaddingMixinType = TPaddingSingleWidgetSlotMixin<TSingleWidgetChildrenWithBasicLayoutSlot
<InPaddingInvalidationRease>, InPaddingInvalidationReason>;
    
    using AlignmentMixinType = TAlignmentSingleWidgetSlotMixin<TSingleWidgetChildrenWithBasicLayoutSlot
<InPaddingInvalidationRease>>;
}
```



这里涉及到好多模板类，第一个是TSingleWidgetChildrenWithBasicLayoutSlot，继承

**TSingleWidgetChildrenWithSlot，**//单个widget儿子和槽

**TPaddingSingleWidgetSlotMixin，**//padding单个widget槽混合

**TAlignmentSingleWidgetSlotMixin。**//对齐单个widget槽混合



![](Image/TSingleWidgetChildrenWithBaseLayoutSlot/image-20221128003717871.png)

然后是这两个宏，比较关键。

```c++
#define SLATE_SLOT_BEGIN_ARGS_TwoMixins(SlotType, SlotParentType, Mixin1) \
	public: \
	struct FSlotArguments : public SlotParentType::FSlotArguments, public Mixin1::FSlotArgumentsMixin\
	SLATE_PRIVATE_SLOT_BEGIN_ARGS(SlotType, SlotParentType)

#define SLATE_PRIVATE_SLOT_BEGIN_ARGS(SlotType, SlotParentType)\
	{\
		using WidgetArgsType = SlotType::FSlotArguments;
		using SlotParentType::FSlotArguments::FSlotArguments;
```



```c++
/*
使用这个宏在SLATE_BEGIN_ARGS和SLATE_END_ARGS之间，目的是添加支持，对于槽的构造样式
*/
#define SLATE_SLOT_ARGUMENT(SlotType, SlotName)\
	TArray<typename SlotType::FSlotArguments> _##SlotName;\
	WidgetArgsType& operator + (typename SlotType::FSlotArguments& SlotToAdd) \
	{ \
		_##SlotName.Add(MoveTemp(SlotToAdd));\
		return static_cast<WidgetArgsType*>(this)->Me();\
	} \
	WidgetArgsType& operator + (typename SlotType::FSlotArguments&& SlotToAdd) \
	{ \
		_##SlotName.Add(MoveTemp(SlotToAdd));\
		return static_cast<WidgetArgsType*>(this)->Me();\
	} \

//注意，这里有个右值引用
```



# TSingleWidgetChildrenWithSlot

```c++
//一个FChildren，只有一个儿子，并且可以输入一个模板化的槽
template<typename SlotType>
class TSingleWidgetChildrenWithSlot : public FChildren, protected TSlotBase<SlotType>
```































