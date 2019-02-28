# 第一章

## 1.1 .STL六大组件

### 1.1.1.容器

### 	存放数据的数据结构，vector，list，deque，set，map

### 1.1.2.算法

​	sort,search,copy,erase等

### 1.1.3迭代器

​	是容器和算法之间的胶合剂，一种“泛型指针“。原生指针也是一种迭代器

### 1.1.4.仿函数

​	它是一种重载了operator()的class或class template，一般函数指针可视为狭义的仿函数

### 1.1.5.配接器(adapters)

​	一种用来修饰容器或仿函数或迭代器接口的东西。如queue和stack是deque的adapters

### 1.1.6配置器

​	负责空间配置与管理	



# 第二章

## 2.1.set_new_handler(0)的作用

set_new_handler(0)主要是为了卸载目前的内存分配异常处理函数，这样一来一旦分配内存失败的话，C++就会强制性抛出std:bad_alloc异常，而不是跑到处理某个异常处理函数去处理

## 2.2.ptrdiff_t类型的作用

描述了两个指针之间的距离，如两个（int*）指针相减得到值的 类型就是个ptrdiff_t。两个指针之间的+ -运算后的结果就不再是一个指针了，而是一个“距离”概念，总要有一个类型与之对应吧，为了规范和一致性（可移植），C++定义了ptrdiff_t（就是long int类型）。

## 2.3.allocator对对象操作的顺序

创建对象，先new在construct；删除对象，先destroy再delete。原因：alloc把内存配置和对象构造的操作分开，分别由**alloc::allocate()**（对应operator new）和**::construct()**（对应定位new表达式 ）负责，同样内存释放和对象析够操作也被分开分别由**alloc::deallocate()**和::destroy()负责。这样可以保证高效，因为对于内存分配释放和构造析够可以**根据具体类型(type traits)进行优化。比如一些类型可以直接使用高效的memset来初始化或者忽略一些析构函数。对于内存分配alloc也提供了2级分配器来应对不同情况的内存分配。

## 2.4.typedef说明

```c++
template <class T>
class allocator{
  public:
    typedef T value_type;
    typedef T* pointer;
    typedef const T* const_pointer;
    typedef T& reference;
    typedef const T& const_reference;
    typedef size_t size_type;
    typedef ptrdiff_t difference_type;
};
```

如上图，利用几个typedef将该类它该有的所有类型转换为一个个统一的名字，后面几章要提到的STL里的所有类都是这么做的，这样可以将标准统一，增加代码复用性。（就是为了方便，安全，可复用性等）（网上说法：为什么有这么多的typedef？你可能认为pointer是多余的：它就是value_type*。绝大部份时候这是对的，但你可能有时候想定义非传统的allocator，它的pointer是一个pointer-like的class，或非标的厂商特定类型value_type__far *）

## 2.5SGI空间配置的特征

SGI配置器的名字叫做alloc而不是allocator,同时不接受任何参数，如果你在程序中药采用SGI配置器，那么格式为：

```c++
应该为：
vector<int,std::alloc> iv;
而不是
vector<int,std::allocator<int> > iv;
```

## 2.6.泛化和特化

STL源码中主要是用到了模块的泛化和特化来实现，泛化来应对一般场景的应用，使得代码复用，而特化一般是为了在某种情况下可以优化处理而存在的

## 2.7.<memory>

这个文件包括<stl_contruct.h>:两种全局函数，contruct,destroy

​		     <stl_alloc.h>：定义了第一，二级配置器，两种配置器全重命名为alloc

​		      <stl_uninitialized.h>：三种全局函数：用来填充或复制大块内存数据,un_initialized_copy,un_initialized_fill,un_initialzed_fill_n

## 2.8.stl_construct.h代码说明

1.首先由于在STL里面创建对象都是先分配空间，在构造，反之亦然；在stl_construct中只包含了构造和析构过程

2.先说构造，构造比较简单，只涉及到两层，因为它没有考虑到构造对象数组的情况，第一层为：

```c++
//有参的构造
inline void construct(_T1* __p, const _T2& __value)
//无参的默认构造
inline void construct(_T1* __p)
```

第一层的作用相当于给外部提供的接口

第二层：

```c++
inline void _Construct(_T1* __p, const _T2& __value)
inline void _Contruct(_T1* __p)
```

第二层为两种类型构造函数的具体实现

3.在说析构函数，析构函数层次比较多，横跨四层（由于有对对象数组的析构）

第一层类似于构造函数第一层：

```c++
//析构单一的对象
inline void destroy(_Tp* __pointer)
//析构对象数组
inline void destroy(_ForwardIterator __first, _ForwardIterator __last)
```

它也是提供一种接口的作用

第二层：

```c++
inline void destroy(_Tp* __pointer)
inline void destroy(_ForwardIterator __first, _ForwardIterator __last)
```

第二层有一些特化的函数，提高性能

第三层：

```c++
//_Tp类型是上层传入的__first的value_type
inline void __destroy(_ForwardIterator __first, _ForwardIterator __last, _Tp*)
```

第三层会获取_Tp类型，也就是__first的value_type，它的_Trivial_destructor,看该类型是否有一个比较重要的析构函数（重要的析构函数，比如类中有个指针类型的成员，用完需要delete，那么它的析构函数就是not trivial，重要的）

第四层：

```c++
//1.没有重要的析构函数,函数里面不需要做啥
inline void __destroy_aux(_ForwardIterator,_ForwardIterator,__true_type){}
//2.有重要的析构函数，函数里面需要循环调用destroy，又回到第一层的第一个接口
template <class _ForwardIterator>
inline void __destroy_aux(_ForwardIterator __first, _ForwardIterator __last, __false_type){...}
```

4.STL源码书上的只有三层，相当于去掉了这里面的第一层

5.由于c++本身并不支持对“指针所指之物”的类型判断，所以有value_type；同时也不支持对“对象析构是否为trivial”的判断，所以有了__type_traits<>

## 2.9.stl_alloc.h代码说明

1.为了较少小型区块可能造成的内存破碎问题，SGI设计了双层级配置器；第一级配置器（__malloc_alloc_template）直接使用malloc和free,第二级配置器（__default_alloc_template）则视不同的策略，它相当于一个小型的内存池，维护一个16个有链表

2.两级配置器使用规则：看是否定义__USE_MALLOC字段。若定义了，使用第一级配置器；若没定义使用第二级配置器（SGI STL没定义__USE_MALLOC字段）。此外若使用第二级配置器，其需求的区块大于128B，就会转去调用第一级配置器。有选择的将两种配置器typedef成alloc，将alloc暴露给外层（容器，vector等）使用，它屏蔽了两种不同的配置器。在具体使用中，stl在它的上层又搭了一层，它只提供多少字节的构造，所以在上层加入元素类型的概念（simple_alloc），更易于使用

3.第一级配置器：__malloc_alloc_template

```c++
typedef __malloc_alloc_template<0> malloc_alloc;
typedef malloc_alloc alloc;
```

它对空间的管理是直接调用c语言原生的函数，

allocate中调用malloc，失败的话调用_S_oom_malloc(它里面做了，先释放空间在企图用malloc分配空间，形成一个闭环)

reallocate中调用realloc,失败则调用_S_oom_realloc（它里面先释放空间，然后用realloc重新分配，形成一个闭环）

deallocate中调用free,由于释放一般不会出问题，所以没有出错处理

在这里实现了类似c++ new-handler机制（对内存的配置，释放，重配置，即失败后在重分），它不能直接使用c++ new-handler，因为它是使用malloc而不是使用::operator new来配置内存的

4.第二级配置器：__default_alloc_template

内存池用一个自由链表（free-list）来维护，总共16个free-list，每个链表管理内存块的大小为8的倍数（依次为8,16，24,32,40,48,56...120,128）

free-list的节点结构

```c++
//sizeof(union obj)=4
union obj{
  union obj* free_list_link;
  char client_data[1];//供用户使用，在将来可能指向8字节，16字节等更大空间的地方
};
//如何使用，例子，如果这是一个8字节的free-list
//首先经过一系列的malloc生成
obj* obj1=(obj*)malloc(8);
obj* obj2=(obj*)malloc(8);
//形成free-list
obj1->free_list_link = obj2;
obj2->free_list_link = NULL;
//从free-list中取出一个内存块给用户使用，假设取obj2
obj1->free_list_link = NULL;
//用户使用obj2(这个时候通过client_data字段访问)
obj2->client_data[0] = 'd';
obj2->client_data[7] = 'e';
```

通过上例的说明，free_list_link字段仅仅是4字节，为了指示下一块的位置；而client_data是为了访问当前内存块的所有空间，它是可变的，因为分配出来的可能是8字节，16字节等。

若想形成16个free-list则用一个保存元素为obj*的数组即可，如obj* * free_lists[16];要将8字节free_list的放入，即free_lists[0] = obj1;free_lisst[1]中放的即是16字节的free_list

![1541494263122](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1541494263122.png)

enum在声明变量后不给变量名，也可直接访问里面的成员

第二级配置器的各函数说明

```c++
allocate函数说明：
首先它先判断区块大小，若大于128字节，那么就调用第一级配置器的allocate；若小于，则找到对应的free_list。如果该free_list有可用的区块，那么直接使用，如果没有，那么先将区块大小调至8倍数边界，然后调用refill为该free_list重新填充空间。它是取链表头部的区块
```

```
deallocate函数说明：
判断区块大小，若大于调用第一级配置器的deallocate，反之找到对应的free_list，然后回收。它将区块放回链表头部
```

```
refill函数说明：
首先调用chunk_alloc从内存中取出一个chunk，这个chunk的大小，默认的取20个新区块，chunk大小为20*区块大小（n）;不过内存池中空间不够的话，区块数会少于20.
取出来的20个区块，第一块返给调用者，余下的19块利用循环将其链接在对应的free_list上面
```

```
chunk_alloc函数说明：这个函数主要进行内存池的管理
1.判断所需chunk大小与内存池剩余空间。若小于。就正常处理；若还够提供一个块以上的空间，那么也正常处理，只是nobjs的值会视实际情况而定；最后一块的空间都无法提供，它会首先将这里残余的空间放到合适的free_list中（由于都是8的倍数所以一定能放进去），之后再调用malloc请求分配空间，请求大小为2倍的需求的总空间+heap_size右移四位取上界（headp_size为历来向系统索要内存空间的总和，它是为了适合要求，一个随着配置次数增加而愈来愈大的附加值，由于heap_size在第3步变化，由于heap_size至少是需求空间的两倍，所以右移四位之后仍能保持是8的倍数）
2.若成功malloc成功，则直接到第3步；若上步malloc失败，则说明系统堆空间不足，然后从比请求块要大的free_list获取一个区块（若下一个free_list没有则再下一个），之后再调用chunk_alloc（这时能保证一定会调用成功，肯定是前两种情况），返回用户合适的chunk指针和nobjs；如果还是比较倒霉，free_list也没有了，它就会调用第一级配置器的allocate，因为在这里面有对于out-of-memory的处理，即malloc失败后会重复的malloc，最后终于又获得了想要的内存空间了
3.先将heap_size加上新申请的空间，它就调用chunk_alloc，返回用户合适的chunk指针和nobjs

```



> ps：这个分配器存在一定性能缺陷，在多线程的情况下，在allocate的时候，它会给free_list加锁，只有单个线程才能分配，而像gcc的stl，它是利用了je malloc当做它的分配器，而je malloc的实现中，是让每个线程都拥有自己的free-list

## 2.10.stl_uninitialized_.h代码说明:

由于拷贝构造和拷贝赋值的效率存在差异（初始化列表），所以就有了__uninitialized_fill_n和fill_n等

fill用来处理给它初始化n个相同值的情况，copy反之

```c++
uninitialized_fill_n函数说明：
//第一层，用__VALUE_TYPE得到__x的类型
template <class _ForwardIter,class _Size,class _Tp>
inline _ForwardIter uninitialized_fill_n(_ForwardIter __first, _Size __n,const _Tp& __x) 
//第二层,_Tp1为__x的真实类型，使用__type_traits萃取出is_POD_type
template <class _ForwardIter,class _Size,class _Tp,class _Tp1>
inline _ForwardIter __uninitialized_fill_n(_ForwardIter __first, _Size __n,const _Tp& __x, _Tp1*)
//第三层
//若是POD类型，POD类型为标量类型或传统的C struct类型，POD型别拥有不重要的构造，析构函数，使用STL算法的fill_n
template <class _ForwardIter,class _Size,class _Tp>
inline _ForwardIter __uninitialized_fill_n_aux(_ForwardIter __first, _Size __n, const _TP& __x, __true_type)
//若不是POD类型，循环调用_Contruct构造__result及它之后的空间
template <class _ForwardIter, class _Size, class _Tp>
inline _ForwardIter __uninitialized_fill_n_aux(_ForwardIter __first, _Size __n, const _TP& __x, __false_type)

```

```c++
uninitialized_copy函数说明：类似于上一个函数
//第一层  
template <class _InputIter,class _ForwardIter>
inline _ForwardIter uninitialized_copy(_InputIter __first, _InputIter __last, _ForwardIter __rsult)
//第二层
template <class _InputIter,class _ForwardIter,class _Tp>
inline _ForwardIter __uninitlialized_copy(_InputIter __first, _InputIter __last, _ForwardIter __result,_Tp*)
//第三层,
//若是POD类型,是用stl算法的copy
template <class _InputIter,class _ForwardIter>
inline _ForwardIter __uninitlialized_copy_aux(_InputIter __first, _InputIter __last, _ForwardIter __result, __true_type)
 //若不是POD类型,循环调用_Contruct构造__result及它之后的空间
template <class _InputIter, class _ForwardIter>
inline _ForwardIter __uninitlialized_copy_aux(_InputIter __first, _InputIter __last, _ForwardIter __result, __true_type)
    
  
    //两个特例 char*和wchar_t，在内部使用memmove(注：memmove函数的功能同memcpy基本一致，但是当src区域和dst内存区域重叠时，memcpy可能会出现错误，而memmove能正确进行拷贝。)
inline char* uninitialized_copy(const char* __first, const char* __last, char* __result)
inline wchar_t* uninitialized_copy(const wchar_t* __first, const wchar_t* __last, wchar_t* __result)
    

```

```
uninitialized_fill函数说明：
```

uninitialized_fill_n和uninitialized_fill，uninitialized_copy_n和uninitialized_copy这两组没有本质的区别，只是参数不一样，如uninitialized_fill_n是使用__first和__N来指示源容器的范围，然后__x为填充的值，而同样uninitialized_fill使用__first和__last来指示源容器的范围，然后__x为填充值。另外一组是同理



# 第三章  迭代器概念与traits编程技法

## 3.1.iterator模式的定义

提供一种方法，使之能够依序巡防某个聚合物（容器）所含的各个元素，而又无需暴露该聚合物的内部表达方式

## 3.2 每一种STL容器都提供专属迭代器的原因？

如果自己设计的话，会暴露容器实现的细节（如该迭代器指向的容器里元素的类型，容器的获取下一个元素的操作），同时还得对实现的细节有十分了解。由于无法避免这些问题，那么就将迭代器的开发有每种容器的设计者自己来设计

## 3.3关于迭代器类型推导

c++不提供在运行的时候类型推导（c++只支持sizeof,不支持typeof）

但是其中一个办法可以利用模块来进行在编译时推导，如

```c++
int main(){
    int i;
    func(&i);
}

template <class I>
    inline void func(I iter){
    func_impl(iter,*iter);
}

template<class I,class T>
    inline void func_impl(I iter,T t){
    T tmp;//到这里该迭代器所指的类型已经被解析出来了，即为T
    ....
}

```

但是这个方法不是万能的（比如要推测的类型是函数的返回值）。最常用的类型有五种，在某些情况下，上述办法没用，所以就诞生了Traits编程技法。trait编程的核心思想就是：声明内嵌类型。trait解决函数返回值类型推导问题，如下

```c++
//traits版本1
template <class T>
struct MyIter{
  typedef T value_type;//traits编程  
    T* ptr;
    MyIter(T* p=0):ptr(p){}
    T& operator*()const{return *ptr};
    //...
};

template <class I>
    typename I::value_type fucn(I ite){
    return *ite;
}

int main(){
    MyIter<int> ite(new int(8));
    cout << func(ite);
}
```

上述代码就可以满足在函数返回值里推测类型，func函数返回值中药加typename的原因：因为T是一个template参数，在它别编译器具体化之前，编译器对T一无所知，编译器不知道MyIter<T>::value_type是一个类型还是一个member function或是一个data member.所以关键字typename的作用是告诉编译器这是一个类型，这样可以顺利通过编译（推测这个编译通过应该是编译的前半段，判断这个模块函数声明是否有效；之后才是根据对模板函数的使用来实例化具体的函数）。

ps:traits编程还有个漏洞，并不是所有的迭代器都是class type,比如原生的指针，无法为他定义内嵌类型。为了将指针也包括进去，所以就出现了偏特化（partial specialization）

```c++
//traits版本2
//接上版本1代码
template <class I>
    struct iterator_traits{
        typedef typename I::value_type value_type;
    }
//将版本1的函数升级
template <class I>
    typename iterator_traits<I>::value_type func(I ite){
        return *ite;
    }
```

在版本2中多了一个中间层，目的在于是这个就可以出现特化版本了（注：偏特化的字面意思容易误解，偏特化不一定是对模板参数T或V或U指定某个参数，可以是提供另一份template定义式），如下

```c++
template <class T>
    struct Iterator_traits<T*>{
        typedef T value_type;
    };
//如Iterator_traits<int*>::value_type 得到的template T为int,那么value_type也为int

//其实利用版本1也可以实现,将函数模板偏特化，但是这样做十分不好，这样就把实现细节交给用户，使用Iterator_traits用户使用起来很舒服，一个操作屏蔽底层多个实现
template <class I*>
    typename I func(I* ite){
        return *ite;
    }

//若出现Iterator_traits<const int*>::value_type 得到的value_type为const int,不能用得到类型声明一个变值，来满足要求，所以就需要一个新的偏特化类型
template <class T>
    struct Iterator_traits<const T*>{
        typedef T value_type;
    };
```

 

## 3.4.traits的作用

它可以萃取出每个迭代器相应的类型。同时为了让traits能够正常运行，每个迭代器必须遵循约定，自行以内嵌型别定义的方式定义出相应型别。

```c++
//由于常用的迭代器相应的类型有五种:value type,difference type,pointer,reference,iterator catagoly,所以迭代器萃取器的定义如下（当然对于迭代器是元素指针，它会提供相应的偏特化版本）
template <class I>
    struct iterator_traits{
      	typedef typename I::iterator_category iterator_category;
        typedef typename I::value_type value_type;
        typedef typename I::difference_type difference_type;
        typedef typename I::pointer pointer;
        typedef typename I::reference reference;
    };

//举例，vector嵌套型别定义
template <class T,class Alloc=alloc>
class vector{
	public:
		typedef T value_type;  //
		typedef value_type* pointer;//
		typedef value_type* iterator;
		typedef value_type& reference;//
		typedef size_t size_type;
		typedef ptrdiff_t difference_type;//
};
//如上，vector中嵌套型别定义有4处和iterator_traits四处相同，除了iterator_category，该类型应该是描述迭代器的自身属性的，可读可写等


```



## 3.5.迭代器的五种型别 

### 3.5.1 value_type

> 描述迭代器所指对象的型别

### 3.5.2 difference_type

> 它是用来表示两个迭代器之间的距离，因此可以表示一个容器的最大容量，其实这个值实际表示的**是两个迭代器相隔元素的个数**。它与容器中的size_type的区别在于，difference_type的值可负可正，而size_type只能是负数                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                          

### 3.5.3 reference_type

> 迭代器所指对象的引用

### 3.5.4 pointer

> 迭代器所指对象的指针

### 3.5.5 iterator_category

> 迭代器的种类
>
> 1.Input Iterator:只读
>
> 2.Output Iterator:只写
>
> 3.Forward Iterator:容许“写入型”算法在此种迭代器形成的区间上进行读写操作
>
> 4.Biderectional Iterator:可双向移动，这样使得某些算法可以逆向遍历迭代器区间
>
> 5.Random Access Iterator:前四种迭代器中提供一部分指针算术能力（1,2,3支持operator++,4还可以支持operator--）,这一种涵盖了所有的指针算法能力，比如，p+n,p-n,p[n],p1-p2,p1<p2。

​	区分不同迭代器类型的目的在于在设计算法的时候，可以针对能力更强大的迭代器烈性提供更高效的算法。在算法看来，迭代器相当于容器的抽象化了，所有容器抽象成了五种类型

```
                          第一层：     算法
                          第二层：     迭代器 //它为一个适配层
                          第三层：     容器
```

```
五种迭代器之间的从属关系
			Input Iterator         output Iterator
			     ↘                      ↙
                       Forward Iterator
                              ↓
                        Bidirecttional Iterator
                             ↓
                        Random Access Iterator
```

```c++

```

## 3.6 stl_iterator.h>代码分析

无

## 3.7 SGI STL的私房菜：__type_tratis

> 前面谈的iterator_traits是一种类型的traits编程，这是为了萃取迭代器的特性；为了萃取类型（type）的特性SGI提供的__type_traits。双底线是指这个SGI STL内部使用的，不在STL标准的范围内

```c++
struct __true_type{};
struct __false_type{};

template <class _Tp>
struct __type_traits{
	typedef __true_type this_dummy_member_must_be_first;
	
	typedef false_type has_trivial_default_constructor;
	typedef false_type has_trivial_copy_constructor;
	typedef false_type has_trivial_assignment_operator;
	typedef false_type has_trivial_destructor;
	typedef false_type is_POD_type;
};
```

注：使用__true_type和__false_type而不用bool型数值区分的原因，前面在编译时就可以区分是那种类型，调用特定的函数了，而后面的运行时才能确定怎么运行，效率更高。

它包所有内嵌类型都定义为__false_type的目的：使用最保守的值。然后对于标量类型（char,short,int,long ,float,double）设计适当的__type_traits的特化版本就行



# 第四章 序列式容器

## 问题：

在普通局部对象的创建和删除中，构造函数和析构函数并没有对空间的操作？那是怎么创建和释放空间的

答：主要是编译器来做。编译在遇到局部对象的定义式的时候，就先插入在栈上分配空间，然后调用对象的构造函数，然后在超出对象的作用域之后，编译器先调用对象的析构函数，然后在释放空间（不能显示的调用局部对象的析构函数，然后会出现调用两次析构函数的情况，可能会出错）。最后在STL的负责析构的全局函数destroy里面调用析构函数的理由是，这个函数值负责对对象的析构而不负责对空间的释放，同时在stl里面容器的空间都是malloc出来的,详见stl_alloc.h



## 4.1.vector的迭代器类型

由于vector维护的是一个连续线性空间，所以不论元素类型是什么，普通指针都可以作为vector的迭代器。vector支持随机存取，所以提供的是Random Access Iterator

```c++
//如下代码，其提供的svite的类型就是Shape*
vector<Shape>::iterator svite;
```

## 4.2 []和at的区别

> at会判断访问的位置是否越界，[]不会，代码如下

```c++
reference operator[](size_type __n) {
		return *(begin()+ __n);	
	}
	const_reference operator[](size_type __n)const {
		return *(begin() + __n);
	}

#ifdef __STL_THROW_RANGE_ERRORS
	void _M_range_check(size_type __n)const {
		if (__n >= this->size()) {
			__stl_throw_range_error("vector");
		}
	}

	reference at(size_type __n) {
		_M_range_check(__n);
		return (*this)[__n];
	}
	const_reference at(size_type __n)const {
		_M_range_check(__n);
		return (*this)[__n];
	}
#endif
```

## 4.3先分配空间在构造的具体体现

```c++
vector(size_type __n, const _Tp& __value,const allocator_type& __a = allocator_type()):_Base(__n,__a)//_Base申请空间
	{
		//构造
		_M_finish = uninitialized_fill_n(_M_start, __n, __value);
	}
```

## 4.4两类函数

```c++
//未涉及分配空间的函数
iterator begin();
iterator end();
size_type size();
size_type max_size();
size_type capacity();
bool empty();
reference operator[](size_type __n);
reference at(size_type __n);
explicit vector(const allocator_type& __a = allocator_type());//空间为0
~vector();
reference front()；
reference back()；
void swap(vector<_Tp, _Alloc>& __x)；//比较快，只交换了三个iterator(_M_start,_M_finish,_M_end_of_storage)
void pop_back()；
iterator erase(iterator __position)；iterator erase(iterator __first, iterator __last)；
void clear()；
//涉及分配空间的函数
vector(size_type __n, const _Tp& __value,const allocator_type& __a = allocator_type())；
explicit vector(size_type __n)；
vector(const vector<_Tp, _Alloc>& __x)；//拷贝赋值
vector(_InputIterator __first, _InputIterator __last, const allocator_type& __a = allocator_type())；
void reserve(size_type __n)//当__n小于caapacity()时，不做操作；反之，会将原来的空间释放，新分配__n个空间，将原来的值移到新空间
void assign(size_type __n, const _Tp& __val)//里面分了__n,size(),capacity()三个值的比较，只有__n>capacity()会分配空间
void push_back(const _Tp& __x)；void push_back()；//当元素超过容量，会分配空间
iterator insert(iterator __position, const _Tp& __x);//当空间大于容量了，比较原容量和0，若相等，则新容量为1，反之则为原容量的2倍
void resize(size_type __new_size, const _Tp& __x);//当size>大于size()，调用的是insert,其中size大于容量的话，会分配空间
```

最复杂的两个函数：assign和insert

_M_fill_assign:从头开始赋值n个相同值           _M_fill_insert:在一个迭代器指示的位置插入n个相同的值

_M_assign_aux:用迭代器描述的一段区间的数据control头开始赋值      _M_insert_aux:在一个迭代器指示位置插入一个数据

_M_range_insert：在一个迭代器指示的位置插入一段用迭代器描述的区间的数据

```c++
//vector的核心成员变量
class vector :protected _Vector_base<_Tp, _Alloc> {
	using _Base::_M_allocate;//函数
	using _Base::_M_deallocate;//函数
	using _Base::_M_start;//变量
	using _Base::_M_finish;//变量
	using _Base::_M_end_of_storage;//变量
};
```



## 4.5使用迭代器区间进行构造的问题

> 1.函数原型：vector(_InputIterator __first, _InputIterator __last, const allocator_type& __a = allocator_type()) 
>
> 1.1里面分了迭代器是input_iterator_tag还是forward_iterator_tag，但是里面的push_back和用unitialized_copy的区别？
>
> 答：为了能用其他种类的迭代器构造？比如链表是input_iterator_tag只能用push_back
>
> 1.2这个函数模板提供这个特化的目的vector < const _Tp* __first, const _Tp* __last, const allocator_type& __a = allocator_type())
>
> 答：为了能用指针类型构造？

## 4.6.stl_list.h说明

### 4.6.1 list的迭代器

由于list不能像vector一样使用普通指针作为迭代器（不保证空间连续），所以声明自己的迭代器，实现递增递减，取值等操作

### 4.6.2._List_iterator定义的目的和继承体系的说明

> template<class _Tp, class _Ref, class _Ptr>
> struct _List_iterator : public _List_iterator_base_
>
> 如上，它除了定义_Tp还定义了_Ref,和_Ptr,因为不一定一个变量的指针就是_Tp*,比如offset_ptr(映射地址独立指针，boost库里面的，是一种fancy pointer)，它是为了共享内存而设计的指针。

> 同时在 _List_iterator也有个继承体系，
>
> 基类：_List_iterator_base，里面定义了节点的指针和重载了两个运算符==,！=
>
> 子类：_List_iterator,里面重装了四个运算符，++（前置和后置），--（前置和后置）

### 4.6.3._List_node继承体系的说明

> 基类：
>
> ```c++
> struct _List_node_base {
>   _List_node_base* _M_next;
>   _List_node_base* _M_prev;
> };
> ```
>
> 子类：
>
> ```c++
> template <class _Tp>
> struct _List_node : public _List_node_base {
>   _Tp _M_data;
> };
> ```
>
> 这样可以方便于扩展

### 4.6.4._List_base的继承体系的说明

> SGI考虑到兼容性，兼顾了三种类型的分配器，1.普通的符合标准的分配器，2.符合标准的分配器没有非静态数据的分配器，3.SGI风格的分配器
>
> 第一种和第二种是放在一块的，_List_base有个基类 _List_alloc_base模板基类，定义如下_
>
> _template <class _Tp,class _Allocator,bool _IsStatic>//第一种分配分配器，它有两个成员变量，_M_node（节点指针），_Node_allocator（分配器对象）
> class _List_alloc_base_
>
> _而对于二种分配器，它就是上述模板的一个特例
> template <class _Tp,class _Allocator>//它有唯一个成员变量， _M_node（节点指针）
> class _List_alloc_base<_Tp, _Allocator, true> //特化，第二种分配器，也就是有静态的分配器，不需要创建变量了上述两个类提供了和分配空间相关的三个函数：get_allocator()，_M_get_node()，_M_put_node()_
>
> _最后用一个_List_base来继承这个基类模板，
>
> template <class _Tp,class _Alloc>
> class _List_base : public _List_alloc_base<_Tp, _Alloc, _Alloc_traits<_Tp, _Alloc>::_S_instanceless>//它没有成员变量
>
> 在这里面就利用基类的分配空间的处理函数_M_get_node()，_M_put_node()，构建出它的构造函数和析构函数
>
> 第三种：不在上述的继承体系中，直接声明 _List_base模板类
>
> template <class _Tp,class _Alloc>//第三种类型，只有一个成员变量， _M_node（节点指针）
> class _List_base
>
> 它有上面继承体系的函数全放在了一块，三个分配空间函数和构造和析构

> ```c++
> _List_base(const allocator_type&)//允许外部传入分配器，但不是在这起作用
> 	{
> 		_M_node = _M_get_node();//第一个节点不放数据,为了形成前闭后开的区间
> 		_M_node->_M_next = _M_node;
> 		_M_node->_M_prev = _M_node;
> 	}
> ```
>
> 对于他的构造函数会首先初始化出来一个节点，由_M_node指向，是为了统一容器，像vector一样是一个前闭后开的区间，end()出来的迭代器指向一个节点但是没有数据
>

### 4.6.5list相关函数说明

```c++
allocator_type get_allocator();//获取分配器类型
_Node* _M_create_node(const _Tp& __x);//它是protected,相对于其父类的_M_get_node，加上了构造的过程
_Node* _M_create_node();//同上，使用默认构造函数
iterator begin();const_iterator begin()const;
iterator end();const_iterator end()const;
bool empty()const;
size_type size()const;//计算list的大小是每次调用distance来计算list的长度，而不是在list里面专门有个字段来记录长度
reference front()const;const_reference front()const；//取容器第一个元素的引用
reference back();const_reference back()const;//注：它不同于end()，它是end()指向的节点的前一个节点元素的引用
void swap(_List<_Tp, _Alloc>& __x);//交换两个list，它只是交换里面的_M_node变量，所以比较快速
iterator insert(iterator __position, const _Tp& __x);//将__x插入到__position位置，即__position及__position之后的数据都往后移，执行的操作就是双向链表大的插入
void insert(iterator __pos, size_type __n, const _Tp& __x);// 插入连续几个相同的元素
void push_front(const _Tp& __x);//调用insert插入到第一个元素
void push_back(const _Tp& __x);//调用insert插入到最后一个元素的后一个位置，即end()的位置
iterator erase(iterator __postion);//及时双向链表的删除，返回__position的后一个元素的迭代器
void clear();//调用父类(_List_base)的clear
void pop_front();//弹出第一个元素
void pop_back();//弹出最后一个元素，即end()的前一个位置的元素
list(size_type __n, const _Tp& __value,
       const allocator_type& __a = allocator_type());//调用insert插入n个元素
```

### 4.6.6list的核心成员变量

```c++
template <class _Tp, class _Alloc = __STL_DEFAULT_ALLOCATOR(_Tp) >
class list : protected _List_base<_Tp, _Alloc> {
protected:
#ifdef __STL_HAS_NAMESPACES
  using _Base::_M_node;//成员变量
  using _Base::_M_put_node;
  using _Base::_M_get_node;//在list改装成了_M_create_node
#endif /* __STL_HAS_NAMESPACES */
}；
```

### 4.6.7list的sort一些限制

list不能使用STL算法的sort算法，因为这个sort只支持RamdonAccessIterator，所以它自己实现了一个sort,使用的是快排

### 4.6.8关于函数transfer

它的作用是将某个连续范围的元素迁移到某个特定位置之前

然后它就成为了一些函数的基础，如，splice(作用同transfer),reverse,merge,sort

### **4.6.8SGI的list一个逻辑bug**

> 由于list为了实现前闭后开的语义，那么它在初始化的时候（不给它添加元素），它也会创建出一个节点出来（头节点），但是这个时候使用begin()获取第一个元素的迭代器，（按照逻辑来说，现在还有插入元素，应该取不出来，也不能给它赋值，但是！在gcc中这段代码是可以跑通的，这就导致了你认为赋值的第一个元素其实没有赋值）
>
> ```c++
> #inlcude <list>
> 
> int main(){
>     list<int> i;
> 	*i.begin() = 1;
> }
> ```

> 但是同样的代码，在vs上无法跑通，vs的STL逻辑做的更好些，它在代码里面判断了给迭代器取引用的时候，这个迭代器是否是头结点的指针
>
> ```c++
> 	_NODISCARD reference operator*() const
> 		{	// return designated value
>  #if _ITERATOR_DEBUG_LEVEL == 2
> 		const auto _Mycont = static_cast<const _Mylist *>(this->_Getcont());
> 		if (_Mycont == 0
> 			|| this->_Ptr == nullptr
> 			|| this->_Ptr == _Mycont->_Myhead)
> 			{	// report error
> 			_DEBUG_ERROR("list iterator not dereferencable");
> 			}
> 
>  #elif _ITERATOR_DEBUG_LEVEL == 1
> 		_SCL_SECURE_VALIDATE(this->_Ptr != nullptr);
> 		const auto _Mycont = static_cast<const _Mylist *>(this->_Getcont());
> 		_SCL_SECURE_VALIDATE(_Mycont != 0);
> 		_SCL_SECURE_VALIDATE_RANGE(this->_Ptr != _Mycont->_Myhead);
>  #endif /* _ITERATOR_DEBUG_LEVEL */
> 
> 		return (this->_Ptr->_Myval);
> 		}
> ```
>
>

## 4.7deque

> vector是单向开头的连续性空间，deque是一种双向开口的“连续空间”（中间会有中断，它是分段的连续空间组成）它两的区别：deque容许常量时间内对头端进行元素的插入和删除和deque没有容量的概念

> deque虽然也提供Ranmdon Access Iterator，但它的迭代器不是普通指针，所以在排序的时候，我们应该现将deque复制到一个vector中，排好序，再复制回来

> deque利用一块map(不是STL的map容器)作为主控，map是一块连续空间，空间上的每个元素也都是一个指针，指向一块存放元素的连续空间（默认512B，那么元素个数=512/sizeof(_Tp)）

## 4.8 stack

它是一种配接器，默认利用了deque来实现，将其接口改为适合自己特性的接口，它里面 的唯一的成员变量就是用deque声明的变量

> 由于stack只有顶端的元素才能被取用，所以它不提供迭代器

ps:可以更改stack的下层容器，比如list,只需要如下定义即可

```c++
stack<int,list<int> > listStack;
```

## 4.9 queue

它也是一种配接器，类似于stack，默认情况使用deque作为底层的存储容器，不提供迭代器，同时也可以更改底层容器为list

## 4.10 heap

它不是stl的容器，它是作为priority queue的助手而存在的，heap也没有迭代器

heap的实现可由一个array和一组heap算法（用来来插入，删除元素，去极值）。ps:可用vector代替array

### 4.10.1 push_heap算法

> 需要用户显示的将插入元素放到array的尾部，之后执行算法push_back,在依次向上回溯，即比较自己与自己的父节点值的大小，直到找到插入元素应该在的位置

### 4.10.2 pop_heap算法

> 它首先把第一个元素（目标元素）放在最后一个位置，并保留最后一个元素的值，之后第一个元素就成为空洞元素，然后取它的儿子节点中较大的放在空洞元素处，那么该较大儿子处的元素就成为了空洞元素了，重复前面动作，直到该空洞元素处于了叶子节点，将空洞元素的值赋值为保留的最后一个元素的值。但是由于这个过程没有比较保留的最后一个元素的值与儿子节点值的大小，所以并不能保证这个是一个堆的节点，所以需要将最后一次的空洞元素的位置当做插入了一个新的元素，做一次push_heap

> 用户调用完pop_heap后，只需要取出最后一个元素（back()），然后在调用容器的pop_back删除最后一个元素，才算完成整个过程

### 4.10.3 sort_heap算法

> 调用堆中元素个数次pop_heap即可，然后在原来array上元素的顺序就是单调递增有序的了

### 4.10.4 make_heap算法

> 首先让最后一个元素的父节点为根的子树构建成堆，之后循环、依次使得比这个父节点序号小的节点作为根节点，构建成堆，直到这个父节点到了第一个元素，构建成堆结束。

# 4.11 priority_queue

源码位于stl_queue.h中

默认情况下，priority_queue是由max_heap完成的，默认的底层容器是vector(但是在priority_queue对象中并没有vector对象，是由外面穿进来的，它只是在内部修改了它的操作 )

所以它也是一个adapter

# 4.12 slist

对于它的end()返回需要注意，它返回是一个iterator(0)的临时变量所以在用itr!=slist.end()判断是否结束，会导致出错，

应该这样判断(itr != 0)



# 第五章 关联式容器



## 5.1关联式容器的种类

> 标准的STL关联式容器分为set和map以及他们两的衍生物multiset和multimap

> 上面说的容器默认是用红黑树容器实现的。同时还有一个不在标准规格中的关联容器：hash table。同时有着对应的用它作为底层实现的四个容器，hash_set,hash_map,hash_multiset和hash_multimap

## 5.2 RB-tree

定义：

1.每个节点不是红色就是黑色

2.根节点为黑色

3.如果节点为红色，其子节点必须是黑色（限制红色节点数）

4.任一节点至NULL的任何路径，所含之黑节点数必须相同（限制黑色节点数）

根据规则4，新增节点必须为红，然后根据规则3，新增节点的父节点必须为黑；所以如果是其他情况，需要调整颜色和旋转树（新增节点为红，父节点为红）



## 5.3 如何调整

默认 插入节点为红色，其父节点为红色（这个情况有错	）（其中暗含祖父节点为黑色）

情况一：父节点的兄弟节点为黑色，且为外侧插入（即如果父节点在左边就插入在左边）-》父节点和祖父节点做一次向右（默认是父节点在左边）的单旋转，同时更改这两个节点颜色

情况二：父节点的兄弟节点为黑色，且为内侧插入（即如果父节点在左边就插入在右边边）-》先对父节点和插入节点做一次向左（默认父节点在左边）的单旋转，改变原来的祖父节点和插入节点的颜色，然后对祖父节点做一次向右的单旋转（其实它在做完第一次向左的单旋转后，情况就和情况一类似了，但是不是一样的，后面处理也可以一样）

情况三：父节点的兄弟节点为红色，且为外侧插入（同上，同时对于内侧插入，其实这两种的情况的做法是一样的）-》先对父节点和祖父节点做一次向右的单旋转，然后改变插入节点的颜色。如果原来的祖父节点的父节点是黑色节点，那么现在就直接成功了，如果是红色的，见下

情况四：祖父节点的父节点是红色的，那么就回到原点，重新去判断是属于情况一，情况二，还是情况三，递归回溯的求解，直到没有父子节点是连续红色的



由于情况四，处理起来比较复杂，所以为了，避免这个情况三和情况四，有一个从上至下的程序，假设新增一个节点A，那么就沿着A的路径，只要看到某节点X的两个子节点皆为红色，就把这个X节点改为红色，两个子节点改为黑色，如果X的父节点是黑色那么就不要后面的变化，如果X的父节点是红色，那么现在的情况就变成情况一或者情况二了，按照上面的变化就行。这样处理完之后，插入节点节点就不会出现情况三和情况四了，只会是情况一和情况二。

# 6 map

## 6.1 

在map 里只有[]才能直接得到value值，其他的都是得到一个pair类型的对象（即是一个键值对）

## 6.2

map相当于是Rb_tree容器的配接器

## 6.3 map和multimap的区别

只是调用的Rb_tree的函数不一样，multimap能够允许重复，而map不能。比如map调用的是insert_unique()，而multimap调用时insert_equal()

# 7.set

## 7.1 

set相当于Rb_tree容器的配接器

## 7.2 set和multiset的区别

只是调用的Rb_tree的函数不一样，multiset能够允许重复，而set不能。比如set调用的是insert_unique()，而multiset调用时insert_equal()

# 8.hashtable



## 8.1hashtable和二叉搜索树

二叉搜索树具有对数平均时间的效率表现，但是这样的表现构造在一个假设上：输入数据有足够的随机性；而对于hashtable而言没有这样的限制，它在插入。删除和搜寻上具有常数平均时间的表现。

## 8.2 hash_map,hash_set,hash_multimap和hash_multiset

它们都是基于hashtable实现的容器

## 8.3 hashtable的实现

hashtable采用开链法来实现，即由一组一桶节点组成的集合。一桶的节点是一个linked_list，但是并没有用STL的list或slist，而是自己又实现了一个；而保存桶的集合的容器，采用的使用vector来维护，以便有扩展能力。

## 8.4 hashtable和它的迭代器

在hashtable中声明了它的迭代器为它的友元类，因为迭代器需要访问它的私有成员变量（_M_buckets）。这点和其他容器和其对应迭代器的关系不一样

## 8.5 hashtable自己实现的桶（单向链表）和slist的异同

1.slist有一个头结点（不保存数据）而桶没有，第一个节点的指针就直接放在保存桶的vector中，没有节点就直接放NULL

2.slist的begin()返回头结点之后的第一个节点，而对于hashtable它会遍历自己的vector直至找到有元素的桶，然后取它的第一个元素，如果全没元素，则返回end()的迭代器

2.slist的end()返回一个临时的iterator(0)的迭代器，相似的桶也是返回一个临时的iterator(0,this)的迭代器

3.在插入的过程中，slist直接就采用的头插法，而对于某个数据，找到自己对应的桶之后，先遍历桶，如果找到key值相同的元素，若允许插入相同的元素，则将新的值插入在这个节点的后面，反之（不允许），则直接返回；若果将桶节点都遍历完之后，也没找到相同key的节点，那么就将该节点插入到头部，即它的指针放在vector中

## 8.6 hashtable 桶数量变化的条件

> 当新增元素后，元素的总数和现在桶的大小比较，若元素个数比桶的个数多，那么就扩展桶的个数，扩展的方式，利用那28个质数，第一个大于元素数的质数，基本上是上次的2倍的大小。

## 8.7 hash_set和hash_map使用hashtable默认的桶大小

> 默认的值置为了100,根据桶大小定制规则，所以被分配为了193（在一系列质数值第一个比100大的质数）

# 第六章 算法



# 第九章.注解

## 9.1.注1

```
所有的STL容器都不是线程安全的，主要是因为很难去做，或者做了的话会导致很高的性能损害，比如要在容器里查找一个数，其他的线程可能对该容器把这个值改变，或者删除，增加了这个值，另外如果造成了底层重新分配内存的话，那么还会导致迭代器失效，那么就更会有问题。所以，处理线程安全的问题一般是交给用户来做。
一般的STL容器只提供了对同个容器多个读操作，和多不同容器的多个写操作，一个容器只有一个
```



