属性对象。

TAttribute是个模板类。



```c++
/*
	属性对象
*/
template<typename ObjectType>
class TAttribute;

/*
	属性'getter'委托
	ObjectType GetValue() const
	返回attribute的值
*/
DECLARE_DELEGATE_RetVal(ObjectType, FGetter);

FGetter Getter;//一个类型为FGetter的委托，这个委托只能绑定返回值为ObjecType类型的函数

//默认构造
TAttribute()
    :Value()
    ,bIsSet(false)
    ,Getter()
{}

/*
被构造，通过绑定一个直接的函数，将被调用去生成属性的值按需要
在绑定之后，属性将不会有一个值可以被直接地访问，并且代替一个绑定的函数，将总是被调用去生成值

参数 InUserObject，Shared Pointer到类的实例，包含你想去绑定的成员函数。该属性只保留该类的弱指针。
参数 InMethodPtr，成员函数去绑定。函数的结构(返回值，参数，等)必须匹配IBoundAttributeDelegate的定义。
*/

template<class SourceType>
TAttribute(TSharedRef<SourceType> InUserObject, typename FGetter::template TSPMethodDelegate_Const<SourceType>::FMethodPtr InMethodPtr)
:Value()
,bIsSet(true)
,Getter(FGetter::CreateSP(InUserObject, InMethodPtr))
{}

//还有一些其它构造函数，可以通过指针绑定，或者智能指针绑定

//有一些set函数，但是如果使用了set函数，那么之前的委托会解绑

//Get函数会判断有没有委托，如果有，就执行它，去获取值

//有很多Bind函数，不一定需要去从构造函数，构造一个TAttribute
```



也就是说TAttribute，可以绑定一个委托，然后调用Get从这个委托获取值。



**总结：TAttribute可以绑定委托，然后TAttribute尝试去获取值的时候，就会调用委托，进行执行。**





























