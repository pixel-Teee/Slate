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

