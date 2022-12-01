```c++
/*
FChildren是一个接口，必须被实现，通过所有子容器、
它允许迭代任何widget的儿子的列表，不管底层的widget如何去存储它的儿子

FChildren目的是去被返回，通过GetChildren()方法
*/

//数据成员
SWidget* Owner;

FName Name;

//一些虚函数
//返回儿子的数量
virtual int32 Num() const = 0;
//返回在确切的索引的widget的指针
virtual TSharedRef<SWidget> GetChildAt(int32 Index) = 0;
//返回在确切的索引的widget的const指针
virtual TSharedRef<const SWidget> GetChildAt(int32 Index) const = 0;
```



```c++
/*
偶尔地，你可能需要去保持多个具有不同slot要求地离散的儿子的集合
这个数据结构体可以被用来连接多个FChildren在一个单一的访问器下，那么你可以总是返回所有的children
从GetChildren，但是在它们自己的儿子列表中管理它们
*/
FCombinedChildren
 
void AddChildren(FChildren& InLinkedChildren);//添加儿子

virtual int32 Num() const override;//返回儿子的数量，注意，这个是递归的

virtual TSharedRef<SWidget> GetChildAt(int32 Index) override;//注意，这个也是递归的

virtual TSharedRef<const SWidget> GetChildAt(int32 Index) const override;

virtual int32 NumSlot() const;//递归的，会调用children的NumSlot

virtual const FSlotBase& GetSlotAt(int32 ChildIndex) const override;//返回指定的槽

virtual FWidgetRef GetChildRefAt(int32 Index) override;//返回指定widget的ref
    
TArray<FChildren*> LinkedChildren;//持有一个TArray<FChildren*>
```











































