### 3 array

array底层就是一个定长数组，给定长数组加上迭代器相关的东西，就课题让他像一个容器，符合容器的性质。

```cpp
#define _NOEXCEPT noexcept

template<class _Tp, size_t _Size>
struct array {
    // types:
    typedef _Tp value_type;
    typedef value_type &reference;
    typedef value_type *pointer;
    typedef value_type *iterator;
    typedef ptrdiff_t difference_type;

    typedef size_t size_type;

    _Tp __elems_[_Size];

    const value_type *data() const _NOEXCEPT { return __elems_; }

    // iterators:
    iterator begin() _NOEXCEPT { return iterator(data()); }

    iterator end() _NOEXCEPT { return iterator(data() + _Size); }

    reference operator[](size_type __n) _NOEXCEPT { return __elems_[__n]; }

    reference at(size_type __n);
}

template<class _Tp, size_t _Size>
typename array<_Tp, _Size>::reference
array<_Tp, _Size>::at(size_type __n) {
    if (__n >= _Size)
        __throw_out_of_range("array::at");
    return __elems_[__n];
}
```



偏特化版本：(对size为0情况进行处理)

```cpp
template<class _Tp>
struct array<_Tp, 0> {
    // types:
    typedef _Tp value_type;
    typedef value_type &reference;
    typedef value_type *iterator;
    typedef value_type *pointer;
    typedef ptrdiff_t difference_type;

    typedef size_t size_type;

    typedef typename conditional<is_const<_Tp>::value, const char,
            char>::type _CharType;
    struct _ArrayInStructT {
        _Tp __data_[1];
    };
    _ALIGNAS_TYPE(_ArrayInStructT)
    _CharType __elems_[sizeof(_ArrayInStructT)];

    value_type *data() _NOEXCEPT { return reinterpret_cast<value_type *>(__elems_); }
}
```

