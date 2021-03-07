### 3 分配器 allocators



####  C++ 内存配置操作和释放操作

```cpp
class FOO{};
FOO *pf = new FOO;    
delete pf;
```

对于上述代码，其在底层执行内容为：
		line 2：new操作，首先调用：：operator new分配内存 （2）调用Foo::Foo() 构造对象内容 
		line 3：delete操作，首先调用Foo::~Foo()将对象析构 （2）调用::operator delete释放内存

#### operator new() 和 malloc()



