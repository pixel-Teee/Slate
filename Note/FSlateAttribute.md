所有SlateAttribute类型的基类结构体



```c++
struct FSlateAttributeBase
{
/*
不是所有的无效化被支持，通过SlateAttribute。
ChildOrder：SlateAttribute的更新被做在SlatePrepass，我们不可以添加或者移除children在SlatePrepass。
AttributeRegistration：在快速路径，SlateAttribute被更新在一个循环。迭代不可以被修改，当我们循环的时候
*/
template<typename T>
constexpr static bool IsInvalidateWidgetReasonSupported(T Reason)
{
	return false;
}
constexpr static bool IsInvalidateWidgetReasonSupported(EInvalidateWidgetReason Reason)
{
	return (Reason & (EInvalidateWidgetReason::ChildOrder | EInvalidateWidgetReason::AttributeRegistration)) == EInvalidateWidgetReason::None;
}
};
//只有两个枚举值被支持，其余都不支持，使用一个constexpr静态函数来判断
```

