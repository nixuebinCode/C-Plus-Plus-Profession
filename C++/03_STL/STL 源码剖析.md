# 第一章 STL 概论与版本简介

## 1.1 STL 六大组件 功能与运用

STL 提供六大组件：

1. 容器 (containers)

   各种数据结构，如 vector, list, deque, set, map，用来存放数据

2. 算法 (algorithms)

   各种常用算法如 sort, search, copy, erase...

3. 迭代器 (iterators)

   扮演容器与算法之间的胶合剂，是所谓的＂泛型指针”

4. 仿函数 (functors)

   行为类似函数，可作为算法的某种策略。是一种重载了`operator()`的 class 或 class template

5. 配接器 (adapters)

   一种用来修饰容器 (containers) 或仿函数 (functors) 或迭代器 (iterators) 接口的东西。例如，STL 提供的 `queue` 和 `stack`, 虽然看似容器，其实只能算是一种容器配接器，因为它们的底部完全借助 `deque`, 所有操作都由底层的 `deque` 供应

6. 配置器 (allocators)

   负责空间配置与管理。从实现的角度来看，配置器是一个实现了动态空间配置、空间管理、空间释放的 class template

STL 六大组件的交互关系： Container 通过 Allocator 取得数据储存空间， Algorithm 通过 Iterator 存取 Container 内容， Functor 可以协助 Algorithm完成不同的策略变化， Adapter 可以修饰或套接Functor。

 ![image-20220921110259874](images/image-20220921110259874.png)

## 1.2 SGI STL 实现版本

SGI 版本由 Silicon Graphics Computer Systems, Inc. 公司发展，承继HP 版本。SGI 版本被 GCC 采用。你可以在 GCC 的 “include" 子目录下（例如 C:\cygnus\
cygwin-b20\include\g++)找到所有 STL 头文件。

### 1.2.1 SGI STL 文件分布与简介

其众多头文件中，概略可分为五组：

* C++ 标准规范下的 C 头文件（无扩展名），例如 cstdio, cstdlib, cstring...
* C+＋标准程序库中不属于 STL 范畴者，例如 stream, string. ．．相关文件
* STL 标准头文件（无扩展名），例如 vector, deque, list, map, algorithm, functional...
* C++ Standard 定案前， HP 所规范的STL 头文件，例如 vector.h, deque.h, list.h, map.h, algo.h, function.h …
* **<font color='red'>SGI STL 内部文件（STL 真正实现于此）</font>**，例如 stl_vector.h, stl_deque.h, stl_list.h, stl_map.h, stl_algo.h, stl_function.h...

### 1.2.2 SGI STL 的编译器组态设置(configuration)

不同的编译器对 C+＋语言的支持程度不尽相同。作为一个希望具备广泛移植能力的程序库，SGI STL 准备了一个环境组态文件 <stl_config.h> ，其中定义了许多常量，标示某些组态的成立与否。所有 STL 头文件都会直接或间接包含这个组态文件，并以条件式写法，让预处理器 (pre-processor) 根据各个常量决定取舍哪一段程序代码。

## 1.3 可能令你困惑的 C++ 语法

### 1.3.1 临时对象的产生与运用

所谓临时对象，就是一种无名对象（unnamed objects）。刻意制造临时对象的方法是，在型别名称之后直接加一对小括号，并可指定初值，例如 `Shape(3,5)` 或 `int(8)`，其意义相当于调用相应的 constructor 且不指定对象名称。STL 最常将此技巧应用于仿函数（functor）与算法的搭配上，例如：

```c++
template <typename T>
class print{
public:
    void operator()(const T &elem){
        cout << elem << ' ';
    }
};
vector<int> ivec{0, 1, 2, 3, 4, 5};
// print<int>()是一个临时对象，不是一个函数调用操作
for_each(ivec.begin(), ivec.end(), print<int>());
```

### 1.3.2 静态常量整数成员在 class 内部直接初始化

如果 class 内含 cons t static integral data member, 那么根据C+＋标准规格，我们可以在 class 之内直接给予初值。所谓 integral 泛指所有整数型别，不单只是
指 `int` 。例如：

```c++
template <typename T>
class testClass {
public:
    static const int _datai = 5;
	static const long _datal = 3L;
	static const char _datac ='c';
} ;

int main()
{
	cout << testClass<int>::_datai << endl; / / 5
	cout << testClass<int>::_datal << endl; / / 3
	cout << testClass<int>::_datac << endl; / / c
}
```

### 1.3.3 前闭后开区间表示法［）

任何一个 STL 算法，都需要获得由一对迭代器（泛型指针）所标示的区间，用以表示操作范围。这一对迭代器所标示的是个所谓的前闭后开区间，以 [first, last) 表示。也就是说，整个实际范围从 first  开始，直到 last-1 。迭代器 last 所指的是“最后一个元素的下一位置”。

 ![image-20220921141441082](images/image-20220921141441082.png)

# 第二章 空间配置器

整个 STL 的操作对象（所有的数值）都存放在容器之内，而容器一定需要配置空间以置放资料。

## 2.1 空间配置器的标准接口

根据 STL 的规范，以下是 allocator 的必要接口：

```c++
// 各种type
allocator::value_type
allocator::pointer
allocator::const_pointer
allocator::reference
allocator::const_reference
allocator::size_type
allocator::difference_type
allocator: :rebind			// 一个嵌套的 class template。class rebind<U> 拥有唯一成员 other, 那是一个 typedef, 代表 allocator<U>
allocator: :allocator()
allocator::allocator(const allocator&)
template <class U>allocator::allocator(const allocator<U>&)	// 泛化的copy constructor
allocator::~allocator ()
pointer allocator::address(reference x) const	// 返回某个对象的地址。算式 a.address(x) 等同于 &x
const_poin 七er alloca 七or: :address(const_reference x) const		// 返回某个 const 对象的地址
pointer allocator::allocate(size_type n, const void*= 0)	// 配置空间，足以存储 n 个 T 对象
void allocator::deallocate(pointer p, size_type n)			// 归还先前配置的空间
size_type allocator::max_size() const						// 返回可成功配置的最大量
void allocator::construct(pointer p, const T& x)			// 等同于new((const void*) p) T(x)
void allocator: :destroy(pointer p)							// 等同于p->~T()
```

### 2.1.1 设计一个简单的空间配置器， nxb::allocator

```c++
#ifndef NXBALLOC_H
#define NXBALLOC_H

#include <new>		// for placement new
#include <cstddef>	// for ptrdiff_t, size_t
#include <cstdlib>	// for exit()
#include <climits>	// for UINT_MAX
#include <iostream>	// for cerr

using namespace std;

namespace nxb {

template <class T>
inline T* _allocate(ptrdiff_t size, T*) {
	set_new_handler(0);
	T *tmp = (T*)(::operator new((size_t)(size * sizeof(T))));
	if (tmp == 0) {
		cerr << "out of memory" << endl;
		exit(1);
	}
	return tmp;
}

template <class T>
inline void _deallocate(T *buffer) {
	::operator delete(buffer);
}

template <class T1, class T2>
inline void construct(T1* p, const T2& value) {
	new(p) T2(value);
}

template <class T>
inline void destroy(T* p) {
	p->~T();
}

template <typename T>
class allocator {
public:
	typedef T value_type;
	typedef T* pointer;
	typedef const T* const_pointer;
	typedef T& reference;
	typedef const T& const_reference;
	typedef size_t size_type;
	typedef ptrdiff_t difference_type;

	// rebind allocator of type U
	template <class U>
	class rebind {
		typedef allocator<U> other;
	};

	pointer allocate(size_type n, const void* hint = 0) {
		return _allocate((difference_type)n, (pointer)0);
	}

	void deallocate(pointer p, size_type n) { _deallocate(p); }

	void construct(pointer p, const T& value) {
		_construct(p, value);
	}

	void destroy(pointer p) { _destroy(p); }

	pointer address(reference x) { return (pointer)&x; }

	const_pointer const_address(const_reference x) { return (const_pointer)&x; }

	size_type max_size() const { return size_type(UINT_MAX / sizeof(T)); }
};

}	// end of namespace nxb

#endif	// NXBALLOC_H
```

## 2.2 具备次配置力(sub-allocation) 的 SGI 空间配置器

SGI STL 的配置器与众不同，也与标准规范不同，其名称是 alloc 而非 allocator, 而且不接受任何参数。这个事实通常不会给我们带来困扰，因为通常我们使用缺省的空间配置器，很少需要自行指定配置器名称。例如下面的 vector 声明：

```c++
template <class T, class Alloc = alloc>		// 缺省使用 alloc 为配置器
class vector{...};
```

### 2.2.1 SGI 标准的空间配置器， std ::allocator

虽然 SGI 也定义有一个符合部分标准、名为 allocator 的配置器，但 SGI 自己从未用过它，也不建议我们使用。主要原因是效率不佳，只把 C++ 的 `operator new` 和 `operator delete` 做一层薄薄的包装而已。和 2.1.1 中我们自己实现的 allocator 类似。

### 2.2.2 SGI 特殊的空间配置器， std ::alloc

为了精密分工，STL allocator 决定将内存操作和对象操作区分开来：

* 内存配置操作由 `alloc:allocate()` 负责，内存释放操作由 `alloc: :deallocate()`负责
* 对象构造操作由 `construct()` 负责，对象析构操作由 `destroy()` 负责。

 ![image-20220922104840245](images/image-20220922104840245.png)

### 2.2.3 构造和析构基本工具: `construct()` 和 `destroy()`

这两个作为构造、析构之用的函数被设计为**<font color='red'>全局函数</font>**，符合 STL 的规范。此外， STL 还规定配置器必须拥有名为`construct()` 和 `destroy()`的两个成员函数。然而真正在 SGI STL 中大显身手的那个名为 `std: :alloc` 的配置器并未遵守这一规则（稍后可见）。

1. `construct()` 接受一个指针 `p` 和一个初值 `value`, 该函数的用途就是将初值设定到指针所指的空间上。C+＋的 `placement new` 运算子可用来完成这一任务:

   ```c++
   template <class T1, class T2>
   inline void construct(T1* p, const T2& value) {
     new (p) T1(value);
   }
   ```

2. `destroy()`有两个版本：

   * 第一版本接受一个指针，准备将该指针所指之物析构掉。这很简单，直接调用该对象的析构函数即可：

     ```c++
     template <class T>
     inline void destroy(T* pointer) {
         pointer->~T();
     }
     ```

   * 第二版本接受 `first` 和 `last`两个迭代器，准备将 [first,last) 范围内的所有对象析构掉。我们不知道这个范围有多大，万一很大，而每个对象的析构函数都无关痛痒（所谓trivial destructor) ，那么一次次调用这些无关痛痒的析构函数，对效率是一种伤害。因此，这里首先利用 `value_type()` 获得迭代器所指对象的型别，再利用 `_type_traits<T>` 判断该型别的析构函数是否无关痛痒。若是 (`_true_type`)，则什么也不做就结束；若否(`_false_type`) ，这才以循环方式巡访整个范围，并在循环中每经历一个对象就调用第一个版本的 `destroy()`:

     ```c++
     // 找出元素的数值型别
     template <class ForwardIterator>
     inline void destroy(ForwardIterator first, ForwardIterator last) {
       __destroy(first, last, value_type(first));
     }
     
     // 判断元素的数值型别(value type) 是否有t rivial destructor
     template <class ForwardIterator, class T>
     inline void __destroy(ForwardIterator first, ForwardIterator last, T*) {
       typedef typename __type_traits<T>::has_trivial_destructor trivial_destructor;
       __destroy_aux(first, last, trivial_destructor());
     }
     
     // 如果元素的数值型别(value type) 有non-trivial destructor…
     template <class ForwardIterator>
     inline void
     __destroy_aux(ForwardIterator first, ForwardIterator last, __false_type) {
       for ( ; first < last; ++first)
         destroy(&*first);
     }
     
     // 如果元素的数值观别(value type) 有trivial destructor…
     template <class ForwardIterator> 
     inline void __destroy_aux(ForwardIterator, ForwardIterator, __true_type) {}
     ```

### 2.2.4 空间的配置与释放， `std::alloc`

对象构造前的空间配置和对象析构后的空间释放，由<`stl_alloc.h`> 负责，SGI 对此的设计哲学如下：

* 向 system heap 要求空间
* 考虑多线程状态
* 考虑内存不足时的应变措施
* 考虑过多“小型区块”可能造成的内存碎片问题

SGI 以 `malloc()` 和 `free()` 完成内存的配置与释放。考虑到小型区块所可能造成的内存破碎问题， SGI 设计了**<font color='blue'>双层级配置器</font>**，第一级配置器直接使用 `malloc()` 和 `free()`，第二级配置器则视情况采用不同的策略：

当配置区块超过 128 bytes 时，视之为＂足够大”，便调用第一级配置器。当配置区块小于128 bytes 时，视之为“过小”，为了降低额外负担，便采用复杂 memory pool 整理方式，而不再求助于第一级配置器。

其中`＿malloc_alloc_template` 是第一级配置器，`＿default_alloc_template` 是第二级配置器。SGI 将 `alloc` 定义为第一级或第二级配置器，并为它再包装一个接口，使配置器的接口能够符合STL 规格（`alloc` 并不接受任何 template 型别参数）：

```c++
# ifdef _USE_MALLOC
// 令alloc 为第一级配置器
typedef __malloc_alloc_template<0> malloc_alloc;
typedef malloc_alloc alloc; 	
# else
// 令alloc 为第二级配置器
typedef _default_alloc_template<_NODE_ALLOCATOR_THREADS, 0> alloc;
#endif I* ! _USE_MALLOC *I
```

```c++
template<class T, class Alloc>
class simple_alloc {
public:
    static T *allocate(size_t n)
                { return 0 == n? 0 : (T*) Alloc::allocate(n * sizeof (T)); }
    static T *allocate(void)
                { return (T*) Alloc::allocate(sizeof (T)); }
    static void deallocate(T *p, size_t n)
                { if (0 != n) Alloc::deallocate(p, n * sizeof (T)); }
    static void deallocate(T *p)
                { Alloc::deallocate(p, sizeof (T)); }
};
```

这个接口其内部四个成员函数其实都是单纯的转调用，调用传递给配置器（可能是第一级也可能是第二级）的成员函数。SG ISTL 容器全都使用这个`simple_alloc` 接口、例如：

```c++
template <class T, class Alloc = alloc> // 缺省使用 alloc 为配置器
class vector{
protected:
    // 专属之空间配置器，每次配置一个元素大小
    typedef simple_alloc<value_type, Alloc> data_allocator;
    
    void deallocate(){
        if(...)
            data_allocator::deallocate(start, end_of_storage - start)
    }
    ...
}
```

 ![image-20220923092946676](images/image-20220923092946676.png)

### 2.2.5 第一级配置器 `＿malloc_alloc_template` 剖析

```c++
#if 0
#   include <new>
#   define __THROW_BAD_ALLOC throw bad_alloc
#elif !defined(__THROW_BAD_ALLOC)
#   include <iostream.h>
#   define __THROW_BAD_ALLOC cerr << "out of memory" << endl; exit(1)
#endif
// malloc-based allocator. 通常比稍后介绍的 default alloc 速度慢
// 以下是第一级配置器
// 注意，无“template型别参数”。至于“非型别参数”inst，则完全没派上用场
template <int inst>
class __malloc_alloc_template {

private:
// 以下函数将用来处理内存不足的情况
// oom: out of memory
static void *oom_malloc(size_t);
static void *oom_realloc(void *, size_t);
static void (* __malloc_alloc_oom_handler)();

public:

static void * allocate(size_t n)
{
    void *result = malloc(n);					// 第一级配置器直接使用 malloc()
    if (0 == result) result = oom_malloc(n);	// 无法满足需求时，改用oom_malloc()
    return result;
}

static void deallocate(void *p, size_t /* n */)
{
    free(p);	// 第一级配置器直接使用free()
}

static void * reallocate(void *p, size_t /* old_sz */, size_t new_sz)
{
    void * result = realloc(p, new_sz);					// 第一级配置器直接使用realloc()
    if (0 == result) result = oom_realloc(p, new_sz);	// 无法满足需求时，改用oom_realloc()
    return result;
}

// 以下仿真 C++ 的set_new_handler()。换句话说，你可以通过它指定你自己的 out-of-memory handler
static void (* set_malloc_handler(void (*f)()))()
{
    void (* old)() = __malloc_alloc_oom_handler;
    __malloc_alloc_oom_handler = f;
    return(old);
}

};

// malloc_alloc out-of-memory handling 初值为0 。有待客端设定
template <int inst>
void (* __malloc_alloc_template<inst>::__malloc_alloc_oom_handler)() = 0;

template <int inst>
void * __malloc_alloc_template<inst>::oom_malloc(size_t n)
{
    void (* my_malloc_handler)();
    void *result;

    for (;;) {				// 不断尝试释放、配置、再释放、再配置...
        my_malloc_handler = __malloc_alloc_oom_handler;
        if (0 == my_malloc_handler) { __THROW_BAD_ALLOC; }
        (*my_malloc_handler)();	// 调用处理例程，企图释放内存
        result = malloc(n);		//	再次尝试配置内存
        if (result) return(result);
    }
}

template <int inst>
void * __malloc_alloc_template<inst>::oom_realloc(void *p, size_t n)
{
    void (* my_malloc_handler)();
    void *result;

    for (;;) {
        my_malloc_handler = __malloc_alloc_oom_handler;
        if (0 == my_malloc_handler) { __THROW_BAD_ALLOC; }
        (*my_malloc_handler)();
        result = realloc(p, n);
        if (result) return(result);
    }
}
```

第一级配置器以 `malloc()`, `free ()`, `realloc()` 等 C 函数执行实际的内存配置、释放、重配置操作，并实现出类似 C++ new-handler 的机制。

SGI 第一级配置器的 `allocate()` 和 `reallocate()` 都是在调用 `malloc()` 和 `realloc()` 不成功后，改调用 `oom_malloc()` 和 `oom_realloc()`。后两者都有内循环，不断调用“内存不足处理例程”，期望在某次调用之后，获得足够的内存而圆满完成任务。但如果“内存不足处理例程”并未被客端设定，`oom_malloc()` 和`oom_realloc()` 便老实不客气地调用 `_THROW_BAD_ALLOC`，丢出 `bad_alloc` 异常信息，或利用 `exit(1)` 硬生生中止程序。

### 2.2.6 第二级配置器 `__default_alloc_template` 剖析

小额区块带来的其实不仅是内存碎片，配置时的额外负担 (overhead) 也是一个大问题。索求任何一块内存，都得有一些 “税“ 要缴给系统，区块愈小，额外负担所占的比例就愈大，愈显得浪费。

 ![image-20220923095739085](images/image-20220923095739085.png)

SGI 第二级配置器的做法是，当区块小于 128 bytes 时，则以**<font color='blue'>内存池(memory pool) </font>**管理，此法又称为**<font color='blue'>次层配置(sub-allocation)</font>** ：

每次配置一大块内存，并维护对应之**<font color='blue'>自由链表(free-list)</font>** 。下次若再有相同大小的内存需求，就直接从 free-lists 中拨出。如果客端释还小额区块，就由配置器回收到 free-lists 中。为了方便管理，**<font color='red'>SGI 第二级配置器会主动将任何小额区块的内存需求量上调至8 的倍数</font>**（例如客端要求 30 bytes, 就自动调整为 32 bytes)，并维护16 个 free-lists ，各自管理大小分别为 8, 16, 24, 32, 40, 48, 56, 64, 72, 80, 88, 96, 104, 112, 120, 128 bytes 的小额区块。free-lists 的节点结构如下：

```c++
union obj {
	union obj * free_list_link;
   	char client_data[1];    /* The client sees this. */
};
```

注意，上述 `obj` 所用的是 `union`, 由于`union` 之故，从其第一字段观之，`obj` 可被视为一个指针，指向相同形式的另一个obj。从其第二字段观之，`obj` 可被视为一个指针，指向实际区块。一物二用的结果是，不会为了维护链表所必须的指针而造成内存的另一种浪费（我们正在努力节省内存的开销呢）。

下面是第二级配置器的部分实现内容：

```c++
enum {__ALIGN = 8};							// 小型区块的上调边界
enum {__MAX_BYTES = 128};					// 小型区块的上限
enum {__NFREELISTS = __MAX_BYTES/__ALIGN};	// free-lists 个数
// 以下是第二级配置器
// 注意，无“ template 型别参数”，且第二参数完全没派上用场
// 第一参数用于多线程环境下。本书不讨论多线程环境
template <bool threads, int inst>
class __default_alloc_template {
private:
    // ROUND_UP() 将 bytes 上调至 8 的倍数
    static size_t ROUND_UP(size_t bytes) {
        return (((bytes) + __ALIGN-1) & ~(__ALIGN - 1));
    }
private:
    union obj {			// free-lists 的节点构造
        union obj * free_list_link;
        char client_data[1];    /* The client sees this.        */
  	};
private:
    // 16 个free-lists
    static obj * volatile free_list[__NFREELISTS];
    // 以下函数根据区块大小，决定使用第 n 号free-list。n 从 1 起算
    static  size_t FREELIST_INDEX(size_t bytes) {
        return (((bytes) + __ALIGN-1)/__ALIGN - 1);
  	}
    // 返回一个大小为 n 的对象，并可能加入大小为 n 的其它区块到 free list
    static void *refill(size_t n);
    // 配置－大块空间，可容纳 nobjs 个大小为 "size" 的区块
	// 如果配置 nobjs 个区块有所不便， nobjs 可能会降低
	static char *chunk_alloc(size_t size, int &nobjs);
  	// Chunk allocation state
    static char *start_free;	// 内存池起始位置。只在 chunk_alloc() 中变化
	static char *end_free; 		// 内存池结束位置，只在chunk_alloc() 中变化
	static size_t heap_size;
public:
    static void* allocate(size_t n) { /*详述于后*/ }
	static void deallocate(void *p, size_t n) { /*详述于后*/ }
	static void* reallocate(void *p, size_t old_sz, size_t new_sz);	
};

// 以下是 static data member 的定义初初值设定
template <bool threads, int inst>
char *__default_alloc_template<threads, inst>::start_free = 0;

template <bool threads, int inst>
char *__default_alloc_template<threads, inst>::end_free = 0;

template <bool threads, int inst>
size_t __default_alloc_template<threads, inst>::heap_size = 0;

template <bool threads, int inst>
__default_alloc_template<threads, inst>::obj * volatile
__default_alloc_template<threads, inst> ::free_list[__NFREELISTS] = {0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, };
```

### 2.2.7 空间配置函数 `allocate()`

此函数首先判断区块大小，大于128 bytes 就调用第一级配置器，小于128 bytes 就检查对应的 free list：

* 如果 free list 之内有可用的区块，就直接拿来用
* 如果没有可用区块，就将区块大小上调至 8 倍数边界，然后调用 `refill()`，准备为 free list 重新填充空间。`refill()`将于稍后介绍。

```c++
static void * allocate(size_t n)
{
	obj * volatile * my_free_list;
    obj * result;
	
    // 大于128 就调用第一级配置器
    if (n > (size_t) __MAX_BYTES) {
        return(malloc_alloc::allocate(n));
    }
    // 寻找 16 个free lists 中适当的一个
    my_free_list = free_list + FREELIST_INDEX(n);
    result = *my_free_list;
    if (result == 0) {
        // 没找到可用的 free list, 准备重新填充 free list
        void *r = refill(ROUND_UP(n));	// 下节详述
        return r;
    }
    // 调整free list
    *my_free_list = result -> free_list_link;
    return (result);
  };
```

 ![image-20220923103400023](images/image-20220923103400023.png)

> my_free_list 指向第一个可分配区块，将该区块分配之后，my_free_list 需要指向当前管理的区块之中下一个可分配区块（即当前区块的 free_list_link）

### 2.2.8 空间释放函数 `deallocate()`

该函数首先判断区块大小，大于 128 bytes 就调用第一级配置器，小于 128 bytes 就找出对应的 free list, 将区块回收。

```c++
// p不可以是0
static void deallocate(void *p, size_t n)
{
    obj *q = (obj *)p;
    obj * volatile * my_free_list;
	
    // 大于 128 就调用第一级配置器
    if (n > (size_t) __MAX_BYTES) {
        malloc_alloc::deallocate(p, n);
        return;
    }
    // 寻找对应的free list
    my_free_list = free_list + FREELIST_INDEX(n);
    // 调整free list ，回收区块
    q -> free_list_link = *my_free_list;
    *my_free_list = q;
  }
```

 ![image-20220923104636162](images/image-20220923104636162.png)

> my_free_list 指向第一个可分配区块，q 指向当前待回收区块，此操作相当于把 q 所指区块插到 my_free_list 所指区块之前。

### 2.2.9 重新填充 free lists

当 `allocate` 发现 free list 中没有可用区块了时，就调用 `refill()`，准备为 free list 重新填充空间。新的空间将取自内存池（经由 `chunk_alloc()` 完成）。缺省取得 20 个新节点（新区块），但万一内存池空间不足，获得的节点数（区块数）可能小于 20：

```c++
// 返回一个大小为 n 的对象，并且有时候会为适当的 free list 增加节点
// 假设 n 已经适当上调至 8 的倍数

template <bool threads, int inst>
void* __default_alloc_template<threads, inst>::refill(size_t n)
{
    int nobjs = 20;
    // 调用 chunk_alloc()，尝试取得 nobjs 个区块作为 free list 的新节点
	// 注意参数 nobjs 是 pass by reference
    char * chunk = chunk_alloc(n, nobjs);	// 下节详述
    obj * volatile * my_free_list;
    obj * result;
    obj * current_obj, * next_obj;
    int i;
    
	// 如果只获得一个区块，这个区块就分配给调用者用， free list 无新节点
    if (1 == nobjs) return(chunk);
    // 否则准备调整 free list, 纳入新节点
    my_free_list = free_list + FREELIST_INDEX(n);

    // 以下在 chunk 空间内建立 free list
    result = (obj *)chunk;	// 这一块准备返回给客端
    // 以下导引 free list 指向新配置的空间（取自内存池）
    *my_free_list = next_obj = (obj *)(chunk + n);
    // 以下将 free list 的各节点串接起来
    for (i = 1; ; i++) {
        current_obj = next_obj;
        next_obj = (obj *)((char *)next_obj + n);
        if (nobjs - 1 == i) {
            current_obj -> free_list_link = 0;
            break;
        } else {
            current_obj -> free_list_link = next_obj;
        }
      }
    return(result);
}
```

### 2.2.10 内存池 (memory pool)

从内存池中取空间给 free list 使用，是 `chunk_alloc()` 的工作：

```c++
// 假设 size 已经适当上调至 8 的倍数
// 注意参数 nobjs 是 pass by reference
template <bool threads, int inst>
char*
__default_alloc_template<threads, inst>::chunk_alloc(size_t size, int& nobjs)
{
    char * result;
    size_t total_bytes = size * nobjs;
    size_t bytes_left = end_free - start_free;	// 内存池剩余空间

    if (bytes_left >= total_bytes) {
        // 内存池剩余空间完全满足需求量
        result = start_free;
        start_free += total_bytes;
        return(result);
    } else if (bytes_left >= size) {
        // 内存池剩余空间不能完全满足需求量，但足够供应一个（含）以上的区块
        nobjs = bytes_left/size;
        total_bytes = size * nobjs;
        result = start_free;
        start_free += total_bytes;
        return(result);
    } else {
        // 内存池剩余空间连一个区块的大小都无法提供
        size_t bytes_to_get = 2 * total_bytes + ROUND_UP(heap_size >> 4);
        // 以下试着让内存池中的残余零头还有利用价值
        if (bytes_left > 0) {
            // 内存池内还有一些零头，先配给适当的free list
			// 首先寻找适当的free list
            obj * volatile * my_free_list =
                        free_list + FREELIST_INDEX(bytes_left);
            // 调整free list, 将内存池中的残余空间编入
            ((obj *)start_free) -> free_list_link = *my_free_list;
            *my_free_list = (obj *)start_free;
        }
        
        // 配置 heap 空间，用来补充内存池
        start_free = (char *)malloc(bytes_to_get);
        if (0 == start_free) {
            // heap 空间不足， malloc() 失败
            int i;
            obj * __VOLATILE * my_free_list, *p;
            // 试着检视我们手上拥有的东西。这不会造成伤害。我们不打算尝试配置
			// 以下搜寻适当的free list
			// 所谓适当是指“尚有未用区块，且区块够大”之 free list
            for (i = size; i <= __MAX_BYTES; i += __ALIGN) {
                my_free_list = free_list + FREELIST_INDEX(i);
                p = *my_free_list;
                if (0 != p) {	// free list 内尚有未用区块
                    // 调整 free list 以释出未用区块
                    *my_free_list = p -> free_list_link;
                    start_free = (char *)p;
                    end_free = start_free + i;
                    // 递归调用自己，为了修正 nobjs
                    return(chunk_alloc(size, nobjs));
                    // 注意，任何残余零头终将被编人适当的free-list 中备用
                }
       		}
	    	end_free = 0;	// 如果出现意外（山穷水尽，到处都没内存可用了）
            // 调用第一级配置器，看看 out-of-memory 机制能否尽点力
            start_free = (char *)malloc_alloc::allocate(bytes_to_get);
            // 这会导致抛出异常(exception) ，或内存不足的情况获得改善
        }
        heap_size += bytes_to_get;
        end_free = start_free + bytes_to_get;
        return(chunk_alloc(size, nobjs));
    }
}
```

上述的`chunk_alloc()` 函数函数以 `end_free - start_free` 来判断内存池的水量：

* 如果水量充足，就直接调出 20 个区块返回给 free list
* 如果水量不足以提供 20 个区块，但还足够供应一个以上的区块，就拨出这不足 20 个区块的空间出去。这时候其 pass by reference 的 `nobjs` 参数将被修改为实际能够供应的区块数
* 如果内存池连一个区块空间都无法供应，对客端显然无法交待，此时便需利用 `malloc()` 从 heap 中配置内存，为内存池注入活水源头以应付需求。新水量的大小为需求量的两倍，**<font color='red'>再加上一个随着配置次数增加而愈来愈大的附加量</font>**。

举个例子：

1. 假设程序开始，客端就调用 `chunk_alloc (32, 20)`，于是 `malloc` 配置 40 个 32 bytes 区块，其中第1 个交出，另19 个交给 free_list [3] 维护，余 20 个留给内存池。
2. 接下来客端调用 `chunk_alloc (64, 20)` ，此时 free_list [7] 空空如也，必须向内存池要求支持，内存池只够供应 (32*20)/64=10 个 64 bytes 区块，就把这10 个区块返回，第1 个交给客端，余 9 个由free_list[7] 维护。此时内存池全空。
3. 接下来客端再调用 `chunk_alloc(96, 20)` ，此时 free_list[11] 空空如也，必须向内存池要求支持，而内存池此时也是空的，于是以 `malloc` 配置 40+n （附加量）个 96 bytes 区块，其中第 1 个交出，另 19 个交给 free_list[11] 维护，余 20+n （附加量）个区块留给内存池...

 ![image-20220923114025994](images/image-20220923114025994.png)

万一山穷水尽，整个 system heap 空间都不够了，`malloc()` 行动失败，`chunk_alloc()` 就四处寻找有无“尚有未用区块，且区块够大”之 free lists 。找到了就挖一块交出，找不到就调用第一级配置器。第一级配置器其实也是使用 `malloc()` 来配置内存，但它有 out-of-memory 处理机制，或许有机会释放其它的内存拿来此处使用。如果可以，就成功，否则发出 `bad_alloc` 异常。

## 2.3 内存基本处理工具

STL 定义有五个全局函数，作用于未初始化空间上。前两个函数是 2.2.3 节说过的、用于构造的 `construct()` 和用于析构的 `destroy()`，另三个函数是 `uniniialized_copy()`, `uninitialized_fill()`, `uninitialized_fill_n()`。这三个函数都具有 “commit or rollback"  语意：要么产生所有必要的元素，否则就不产生任何元素，如果任何一个 copy constructor 丢出异常，必须析构已产生的所有元素。

### 2.3.1 `uninitialized_copy`

```c++
template <class InputIterator, class ForwardIterator>
ForwardIterator
uninitialized_copy(InputIterator first, InputIterator last, ForwardIterator result);
```

如果作为输出目的地的 [result, result+(last-first)）范围内的每一个迭代器都指向未初始化区域，则 `uninitialized_copy`会使用 copy constructor，给身为输入来源之 [first,last）范围内的每一个对象产生一份复制品，放进输出范围中。

换句话说，针对输入范围内的每一个迭代器 `i`, 该函数会调用:

```c++
construct(&*(result+(i-first)), *i)
```

产生 `*i` 的复制品，放置于输出范围的相对位置上

如果你需要实现一个容器，这样的函数会为你带来很大的帮助，因为容器的全区间构造函数(range constructor) 通常以两个步骤完成：

* 配置内存区块。足以包含范围内的所有元素
* 使用 `uninitialized_copy` 在该内存区块上构造元素

#### 具体实现

```c++
// first 指向输入端的起始位置。
// last 指向输入端的结束位置（前闭后开区间）。
// result 指向输出端（欲初始化空间）的起始处。
template <class InputIterator, class ForwardIterator>
inline ForwardIterator
  uninitialized_copy(InputIterator first, InputIterator last,
                     ForwardIterator result) {
  return __uninitialized_copy(first, last, result, value_type(result));
}

// 这个函数的进行逻辑是，首先萃取出迭代器 result 的 value type，然后判断该型别是否为 POD 型别
// POD 意指 Plain Old Data, 也就是标量型别 (scalar types) 或传统的 C struct 型别
// POD 型别必然拥有 trivial ctor/dtor/copy/ assignment 函数
// 因此，我们可以对POD 型别采用最有效率的复制手法，而对non-POD 型别采取最保险安全的做法：
template <class InputIterator, class ForwardIterator, class T>
inline ForwardIterator
__uninitialized_copy(InputIterator first, InputIterator last,
                     ForwardIterator result, T*) {
  typedef typename __type_traits<T>::is_POD_type is_POD;
  return __uninitialized_copy_aux(first, last, result, is_POD());
}

// 如果 copy construction 等同于 assignment, 而且
// destructor 是trivial, 以下就有效
// 如果是POD 型别，执行流程就会转进到以下函数。这是藉由 function template 的参数推导机制而得
template <class InputIterator, class ForwardIterator>
inline ForwardIterator 
__uninitialized_copy_aux(InputIterator first, InputIterator last,
                         ForwardIterator result,
                         __true_type) {
  return copy(first, last, result);		// 调用 STL 算法 copy()
}

// 如果是non-POD 型别，执行流程就会转进到以下函数
template <class InputIterator, class ForwardIterator>
ForwardIterator 
__uninitialized_copy_aux(InputIterator first, InputIterator last,
                         ForwardIterator result,
                         __false_type) {
  ForwardIterator cur = result;
  for ( ; first != last; ++first, ++cur)
      construct(&*cur, *first);	// 必须一个一个元素地构造，无法批量进行
  return cur;
}

// 针对 char* 和 wchar_t* 两种型别
// 可以采用最具效率的做法 memmove （直接移动内存内容）来执行复制行为

// 以下是针对 const char* 的特化版本
inline char* uninitialized_copy(const char* first, const char* last,
                                char* result) {
  memmove(result, first, last - first);
  return result + (last - first);
}

// 以下是针对 const char* 的特化版本
inline wchar_t* uninitialized_copy(const wchar_t* first, const wchar_t* last,
                                   wchar_t* result) {
  memmove(result, first, sizeof(wchar_t) * (last - first));
  return result + (last - first);
}
```

### 2.3.2 `uninitialized_fill`

```c++
template <class ForwardIterator, class T>
void uninitialized_fill (ForwardIterator first, ForwardIterator last, const T& x);
```

如果 [first, last) 范围内的每个迭代器都指向未初始化的内存，那么 `uninitialized_fill` 会在该范围内产生 `x` （上式第三参数）的复制品。

换句话说， `uninitialized_fill` 会针对操作范围内的每个迭代器 `i` 调用
```c++
construct(&*i, x)
```

在 `i` 所指之处产生 `x` 的复制品。

#### 具体实现

```c++
// first 指向输出端（欲初始化空间）的起始处。
// last 指向输出端（欲初始化空间）的结束处（前闭后开区间）。
// X 表示初值。
template <class ForwardIterator, class T>
inline void uninitialized_fill(ForwardIterator first, ForwardIterator last, 
                               const T& x) {
  __uninitialized_fill(first, last, x, value_type(first));
}

template <class ForwardIterator, class T, class T1>
inline void __uninitialized_fill(ForwardIterator first, ForwardIterator last, 
                                 const T& x, T1*) {
  typedef typename __type_traits<T1>::is_POD_type is_POD;
  __uninitialized_fill_aux(first, last, x, is_POD());
                   
}

template <class ForwardIterator, class T>
inline void
__uninitialized_fill_aux(ForwardIterator first, ForwardIterator last, 
                         const T& x, __true_type)
{
  fill(first, last, x);			// 调用 STL 算法fill()
}

template <class ForwardIterator, class T>
void
__uninitialized_fill_aux(ForwardIterator first, ForwardIterator last, 
                         const T& x, __false_type)
{
  ForwardIterator cur = first;
    for ( ; cur != last; ++cur)
      construct(&*cur, x);
}
```

### 2.3.3 `uninitialized_fill_n`

```c++
template <class ForwardIterator, class Size, class T>
Forwarditerator
uninitialized_fill_n(Forwarditerator first, Size n, const T& x);
```

`uninitialized_fill_n` 会为指定范围内的所有元素设定相同的初值。

如果 [first, first+n) 范围内的每一个迭代器都指向未初始化的内存，那么 `uninitialized_fill_n` 会调用 copy constructor，在该范围内产生 `x` （上式第三参数）的复制品。

也就是说，面对 [first, first+n) 范围内的每个迭代器 `i`, `uninitialized_fill_n` 会调用

```C++
construct(&*i, x)
```

在对应位置处产生 `x` 的复制品。

#### 具体实现

```c++
// frrst 指向欲初始化空间的起始处。
// n 表示欲初始化空间的大小。
// X 表示初值。
template <class ForwardIterator, class Size, class T>
inline ForwardIterator uninitialized_fill_n(ForwardIterator first, Size n,
                                            const T& x) {
  return __uninitialized_fill_n(first, n, x, value_type(first));
}

template <class ForwardIterator, class Size, class T, class T1>
inline ForwardIterator __uninitialized_fill_n(ForwardIterator first, Size n,
                                              const T& x, T1*) {
  typedef typename __type_traits<T1>::is_POD_type is_POD;
  return __uninitialized_fill_n_aux(first, n, x, is_POD());
                                    
}

template <class ForwardIterator, class Size, class T>
inline ForwardIterator
__uninitialized_fill_n_aux(ForwardIterator first, Size n,
                           const T& x, __true_type) {
  return fill_n(first, n, x);	// 交由 STL 算法执行
}

template <class ForwardIterator, class Size, class T>
ForwardIterator
__uninitialized_fill_n_aux(ForwardIterator first, Size n,
                           const T& x, __false_type) {
  ForwardIterator cur = first;
  for ( ; n > 0; --n, ++cur)
      construct(&*cur, x);
  return cur;
}
```

