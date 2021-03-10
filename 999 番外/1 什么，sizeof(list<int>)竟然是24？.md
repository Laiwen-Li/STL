

之前写了这样一段代码，

```cpp
printf("%d | %d \n", sizeof(std::list<int>), sizeof(std::list<long long>));
/*
24 | 24 
*/
```

首先我们需要知道list是个什么东东：

点进std::list<int>，可以看到以下代码，可以看出list这玩意貌似没有成员？但他继承了一个 \_\_list_imp<_Tp, _Alloc>：

```cpp
template <class _Tp, class _Alloc /*= allocator<_Tp>*/>
class _LIBCPP_TEMPLATE_VIS list
    : private __list_imp<_Tp, _Alloc>
{
    ...
}
```

点进\_\_list_imp可以看到这个代码，有一个\_\_end\_，我们离真相又近了一点了，\_\_end\_是\_\_node\_base类型的，可以看到前边有个typedef：

```cpp
template <class _Tp, class _Alloc>
class __list_imp
{
protected:
    typedef _Tp value_type;
    typedef typename __alloc_traits::void_pointer                   __void_pointer;
    typedef __list_node_base<value_type, __void_pointer> __node_base;
    
    __node_base __end_;
    ...
}
```

点进\_\_list_node_base，看看这是个啥玩意。哦豁，发现了两个指针！

```cpp
struct __list_node_base
{
    typedef __list_node_pointer_traits<_Tp, _VoidPtr> _NodeTraits;
    typedef typename _NodeTraits::__node_pointer __node_pointer;
    typedef typename _NodeTraits::__base_pointer __base_pointer;
    typedef typename _NodeTraits::__link_pointer __link_pointer;

    __link_pointer __prev_;
    __link_pointer __next_;
};
```

注意，这是个父结构体，再看看他子结构体的实现，如下：

```cpp
template <class _Tp, class _VoidPtr>
struct __list_node
    : public __list_node_base<_Tp, _VoidPtr>
{
    _Tp __value_;

    typedef __list_node_base<_Tp, _VoidPtr> __base;
    typedef typename __base::__link_pointer __link_pointer;

    _LIBCPP_INLINE_VISIBILITY
    __link_pointer __as_link() {
        return static_cast<__link_pointer>(__base::__self());
    }
};
```

好啊，终于找到了list的数据成员了，两根指针，一个数据，类似于以下形式：

```cpp
template<class T>
struct test {
    T* prev;
    T* next;
    T t;
}
```

但是回到正题，为什么std::list<int>是24呢？

我的机子是64位机，指针是8bytes，int是4个字节，8*2+4 = 20，没毛病，20。

但是输出为什么是24呢？

遇到这种问题，我们一般就需要从编译器角度考虑了（首先可以肯定，这肯定不是我的问题！）

结构体对齐，嗯结构体对齐，这是编译器考虑的范畴。

内存对齐的作用：

> 字节对齐主要是为了提高内存的访问效率，比如intel 32位cpu，每个总线周期都是从偶地址开始读取32位的内存数据，如果数据存放地址不是从偶数开始，则可能出现需要两个总线周期才能读取到想要的数据，因此需要在内存中存放数据时进行对齐。

结论1：一般情况下，**结构体所占的内存大小并非元素本身大小之和。**

结论2：**结构体内存大小应按最大元素大小对齐，如果最大元素大小超过模数，应按模数大小对齐。**

这里的模数就需要编译器来操作啦！

我们试试取消内存对齐试试：

```cpp
#include <list>

template<class T>
struct test {
    T* prev;
    T* next;
    T t;
}__attribute__((__packed__)) ;

int main(){
    printf("%d | %d \n", sizeof(int), sizeof(long long));
    printf("%d | %d \n", sizeof(std::list<int>), sizeof(std::list<long long>));
    printf("%d | %d \n", sizeof(test<int>), sizeof(test<long long>));
}
/*
4 | 8 
24 | 24 
20 | 24
*/
```

可以发现，test<int>已经是20了，说明list<int>为24的原因就是因为内存对齐所致。

内存对齐的相关知识可以参考：https://www.zhihu.com/question/27862634

详细的内存对齐内容，将在以后探讨....

