stl中默认堆为大根堆，大根堆的定义为：

```cpp
priority_queue<int> q;
```

根据源码中的定义，有如下代码：

```cpp
template <class _Tp, class _Container = vector<_Tp>,
          class _Compare = less<typename _Container::value_type> >
class _LIBCPP_TEMPLATE_VIS priority_queue
{
public:
    typedef _Container                               container_type;
    typedef _Compare                                 value_compare;
    typedef typename container_type::value_type      value_type;
    typedef typename container_type::reference       reference;
    typedef typename container_type::size_type       size_type;

protected:
    container_type c;
    value_compare comp;
}
```

从最开头我们可以看出，声明优先队列时，第一参数为类型，第二参数为容器，第三参数为比较函数（默认小于）。

那么问题来了，为什么默认的这个cmp仿函数为小于的堆，是个大根堆（堆顶元素为最大值）？

建堆时都是一步步push()的，查看源码，可以看到如下函数：

```cpp
// 类里边的声明
void push(value_type&& __v);
// push实现
template <class _Tp, class _Container, class _Compare>
inline
void
priority_queue<_Tp, _Container, _Compare>::push(value_type&& __v)
{
    c.push_back(_VSTD::move(__v));
    _VSTD::push_heap(c.begin(), c.end(), comp);
}
```

从上述代码可以看出，优先队列的push操作就是往容器内push_back一个数，然后执行一个push_heap()操作。

查看push_heap()操作，可以看到如下代码：

```cpp
template <class _RandomAccessIterator, class _Compare>
inline _LIBCPP_INLINE_VISIBILITY
void
push_heap(_RandomAccessIterator __first, _RandomAccessIterator __last, _Compare __comp)
{
    typedef typename __comp_ref_type<_Compare>::type _Comp_ref;
    __sift_up<_Comp_ref>(__first, __last, __comp, __last - __first);
}
```

其又掉用了__sift_up()函数，学过堆的小伙伴应该对这个up操作十分熟悉吧！

__sift_up()函数实现如下：

```cpp
template <class _Compare, class _RandomAccessIterator>
void
__sift_up(_RandomAccessIterator __first, _RandomAccessIterator __last, _Compare __comp,
          typename iterator_traits<_RandomAccessIterator>::difference_type __len)
{
    typedef typename iterator_traits<_RandomAccessIterator>::value_type value_type;
    if (__len > 1)
    {
        // ((__len - 1) -1) / 2;
        __len = (__len - 2) / 2;
        _RandomAccessIterator __ptr = __first + __len;
        if (__comp(*__ptr, *--__last))
        {
            value_type __t(_VSTD::move(*__last));
            do
            {
                *__last = _VSTD::move(*__ptr);
                __last = __ptr;
                if (__len == 0)
                    break;
                __len = (__len - 1) / 2;
                __ptr = __first + __len;
            } while (__comp(*__ptr, __t));
            *__last = _VSTD::move(__t);
        }
    }
}

template <class _Tp>
inline _LIBCPP_INLINE_VISIBILITY _LIBCPP_CONSTEXPR
typename remove_reference<_Tp>::type&&
move(_Tp&& __t) _NOEXCEPT
{
    typedef _LIBCPP_NODEBUG_TYPE typename remove_reference<_Tp>::type _Up;
    return static_cast<_Up&&>(__t);
}
```

从__sift_up()函数的代码我们可以看出：这个cmp仿函数时一直传进来了的，而且是根据

\_\_comp(*\_\_ptr, \__t)一直在执行某操作的。

这里就要涉及到heap的up操作了。

heap的up，简而言之，就是把元素和他的父亲节点（/2便是父亲节点，完全二叉树性质）比较，如果符合某性质，就将该节点上移。

以上源码类似于如下代码：

```cpp
void sift-up ( MaxHeap H)
{
    i = H->size;
    item = H->Element [i];
    for ( ; H -> Element [ i/2 ] < item; i /= 2 ) // 与父结点做比较，i / 2 表示的就是父结点的下标
    {
            H -> Element [ i ] = H -> Element [ i/2 ]; // 向下过滤结点
    }
    H -> Element [ i ] = item ; //若for循环完成后,i更新为父节点i，然后将 item 插入
}
```

对于less的话，就是满足小于，则将节点上移，这样就形成了一个大根堆。

所以大小根堆可以以以下方式声明。

```cpp
// 大根堆   
priority_queue<int, vector<int>, less<int>> q;
// 小根堆   
priority_queue<int, vector<int>, greater<int>> q;
```

