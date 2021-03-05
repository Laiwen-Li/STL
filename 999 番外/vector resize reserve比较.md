### vector resize reserve比较

size：Returns the number of elements in the [vector](http://www.cplusplus.com/vector) 目前存在的元素数。即： 元素个数

capacity：**Return size of allocated storage capacity** 容器能存储数据的个数。 即：容器容量

reserve原型：

```cpp
void reserve (size_type n);
```

作用：

Request a change in capacity

Requests that the [vector capacity](http://www.cplusplus.com/vector::capacity) be at least enough to contain *n* elements.

If *n* is greater than the current [vector capacity](http://www.cplusplus.com/vector::capacity), the function causes the container to reallocate its storage increasing its [capacity](http://www.cplusplus.com/vector::capacity) to *n* (or greater).

In all other cases, the function call does not cause a reallocation and the [vector capacity](http://www.cplusplus.com/vector::capacity) is not affected.

This function has no effect on the [vector size](http://www.cplusplus.com/vector::size) and cannot alter its elements.

resize原型：

```cpp
void resize (size_type n);
void resize (size_type n, const value_type& val);
```

作用：

Change size

Resizes the container so that it contains *n* elements.

If *n* is smaller than the current container [size](http://www.cplusplus.com/vector::size), the content is reduced to its first *n* elements, removing those beyond (and destroying them).

If *n* is greater than the current container [size](http://www.cplusplus.com/vector::size), the content is expanded by inserting at the end as many elements as needed to reach a size of *n*. If *val* is specified, the new elements are initialized as copies of *val*, otherwise, they are value-initialized.

If *n* is also greater than the current container [capacity](http://www.cplusplus.com/vector::capacity), an automatic reallocation of the allocated storage space takes place.

Notice that this function changes the actual content of the container by inserting or erasing elements from it.

通过以上说明我们可以得知：

1. resize改变的是容器的大小（包括capacity和size），reserve则是改变的capacity，size没有改变。
2. reserve是容器预留空间，但在空间内不真正创建元素对象，所以在没有添加新的对象之前，不能引用容器内的元素。
3. resize了以后，容器中就有了相关的元素（对象），是可以引用容器内的元素的。

我们通过以下的一个测试函数，来探讨下我们为什么要使用reserve：

```cpp
#include <iostream>
#include <vector>

using namespace std;

void growPushBack(vector<int> &vec){
    int size = 0, cap = 0;
    for(int i = 0; i < 600; i++){
        cap = vec.capacity();
        size = vec.size();
        if(cap == size){
            cout << endl;
            cout << "after push, capacity will mul 2" << endl;
            cout << "before: " << vec.capacity() << " address: " << &*(vec.begin()) << endl;
        }
        vec.push_back(i);
        if(cap == size){
            cout << "after: " << vec.capacity() << " address: " << &*(vec.begin()) << endl;
        }
    }
}

int main(){
    vector<int> vecIntA;
    //声明vector后未使用reserve，直接进行push_back操作
    cout << "Making vecIntA growing:(empty opt)" << endl;
    cout << "size: " << vecIntA.size() << endl;
    cout << "capacity: " << vecIntA.capacity() << endl;
    cout << "begin:" <<&*(vecIntA.begin()) << endl;
    growPushBack(vecIntA);

    cout << "\n\n-----\n\n" << endl;

    vector<int> vecIntB;
    //声明vecIntB后用reserve来执行其容量为50
    vecIntB.reserve(50);
    cout << "Making vecIntB growing:(reserve) " << endl;
    cout << "size: " << vecIntB.size() << endl;
    cout << "capacity: " << vecIntB.capacity() << endl;
    cout << "begin:" <<&*(vecIntB.begin()) << endl;
    growPushBack(vecIntB);

    cout << "\n\n-----\n\n" << endl;

    vector<int> vecIntC;
    //声明vecIntC后用resize来执行其容量为50
    vecIntC.resize(50);
    cout << "Making vecIntC growing: (resize)" << endl;
    cout << "size: " << vecIntC.size() << endl;
    cout << "capacity: " << vecIntC.capacity() << endl;
    cout << "begin:" <<&*(vecIntC.begin()) << endl;
    growPushBack(vecIntC);
    return 0;
}
```

输出结果为：

```cpp
Making vecIntA growing:(empty opt)
size: 0
capacity: 0
begin:0x0

after push, capacity will mul 2
before: 0 address: 0x0
after: 1 address: 0x7fb4a4604080

after push, capacity will mul 2
before: 1 address: 0x7fb4a4604080
after: 2 address: 0x7fb4a4604090

after push, capacity will mul 2
before: 2 address: 0x7fb4a4604090
after: 4 address: 0x7fb4a4604080

after push, capacity will mul 2
before: 4 address: 0x7fb4a4604080
after: 8 address: 0x7fb4a46040a0

after push, capacity will mul 2
before: 8 address: 0x7fb4a46040a0
after: 16 address: 0x7fb4a46040c0

after push, capacity will mul 2
before: 16 address: 0x7fb4a46040c0
after: 32 address: 0x7fb4a4604100

after push, capacity will mul 2
before: 32 address: 0x7fb4a4604100
after: 64 address: 0x7fb4a4604180

after push, capacity will mul 2
before: 64 address: 0x7fb4a4604180
after: 128 address: 0x7fb4a4604280

after push, capacity will mul 2
before: 128 address: 0x7fb4a4604280
after: 256 address: 0x7fb4a5808200


-----


Making vecIntB growing:(reserve) 
size: 0
capacity: 50
begin:0x7fb4a4604180

after push, capacity will mul 2
before: 50 address: 0x7fb4a4604180
after: 100 address: 0x7fb4a4704080

after push, capacity will mul 2
before: 100 address: 0x7fb4a4704080
after: 200 address: 0x7fb4a4704210


-----


Making vecIntC growing: (resize)
size: 50
capacity: 50
begin:0x7fb4a4704080

after push, capacity will mul 2
before: 50 address: 0x7fb4a4704080
after: 100 address: 0x7fb4a4704530

after push, capacity will mul 2
before: 100 address: 0x7fb4a4704530
after: 200 address: 0x7fb4a47046c0

Process finished with exit code 0

```

由运行结果可知，通过push_back操作，容器中的元素数量（size）一直在增加，当容器中的元素个数（size）达到了capacity值时，capacity是呈指数级增大的（2^n）（注意：此处不同编译器是不一样的，有的编译器是增加目前capacity的一半，即乘以1.5；有的是增加目前capacity，即乘以2），当发生扩容操作时，系统会发生以下操作：

1. 开辟2*capacity的新空间
2. 将原vector中的元素拷贝至新地址
3. 释放原vector的capacity空间

根据输出容器的begin迭代器在内存中的地址，可以知道，容器是换了位置的（在内存中的位置发生了变化）。

为了支持快速的随机访问，vector容器的元素以连续方式存放，每一个元素都紧挨着前一个元素存储。设想一下，当vector添加一个元素时，为了满足连续存放这个特性，都需要重新分配空间、拷贝元素、撤销旧空间，这样性能难以接受。因此STL实现者在对vector进行内存分配时，其实际分配的容量要比当前所需的空间多一些。就是说，vector容器预留了一些额外的存储区，用于存放新添加的元素，这样就不必为每个新元素重新分配整个容器的内存空间。

---

push_back的具体实现：

当数组中增加一个元素x的时候，先判断是否还有备用空间；如果还有备用空间，则将当前指针的值设为x，并将当前的指针加1；若备用空间已经用完，如果之前的空间为0，则重新分配大小为1的空间，否则将空间扩容为之前的两倍，然后将旧容器中的值重新拷贝到新空间中，并重新分配起始指针和当前指针。所以使用vector需要注意的一点就是尽量不要动态给它分配空间。而且空间重新分配之后，之前的所有指针都会失效（特别要注意）。

两倍扩容的实现：

```cpp
void push_back(const T& x) {
    if (finish != end_of_storage) { //若当前还有备用空间
      construct(finish, x); //将当前水位的值设为x
      ++finish; //提升水位
    }
    else
      insert_aux(end(), x); 
}
template <class T, class Alloc>
void vector<T, Alloc>::insert_aux(iterator position, const T& x) {
  if (finish != end_of_storage) {
    construct(finish, *(finish - 1));
    ++finish;
    T x_copy = x;
    copy_backward(position, finish - 2, finish - 1);
    *position = x_copy;
  }
  else {
    const size_type old_size = size(); //获取之前数组的大小
    const size_type len = old_size != 0 ? 2 * old_size : 1; //将当前数组的容量扩大为原来的两倍
    iterator new_start = data_allocator::allocate(len); //重新分配新数组的起始迭代器
    iterator new_finish = new_start;
    __STL_TRY {
      new_finish = uninitialized_copy(start, position, new_start); //将旧数组的值重新分配给当前的新数组
      construct(new_finish, x); //将当前数组的水位的值设为x
      ++new_finish; //提升新数组的水位
      new_finish = uninitialized_copy(position, finish, new_finish); //这语句感觉可有可无，因为它根本就不会执行，position即last，而finish也是last
    }
 
#       ifdef  __STL_USE_EXCEPTIONS 
    catch(...) { //如果重新构造的新数组出现异常，则销毁当前新创建的数组，并释放内存空间
      destroy(new_start, new_finish); 
      data_allocator::deallocate(new_start, len);
      throw;
    }
#       endif /* __STL_USE_EXCEPTIONS */
    destroy(begin(), end()); //将旧数组的空间释放掉
    deallocate();
    start = new_start; //new_start记录新数组的起始位置
    finish = new_finish; //重新设置当前水位的指针
    end_of_storage = new_start + len; //设置新数组的容量
  }
}
```

综上，出于性能的考虑，我们在提前可以知道vector size的情况下，可以提前resize或reserve，这样可以减少扩容时拷贝所付出的时间。

