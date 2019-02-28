#    					muduo源码剖析

# 第一部分       安装

## 1.1 muduo依赖的软件和库

必需：

a.cmake >= 2.8 :  sudo yum install cmake

b.boost库： sudo yum install boost boost-devel boost-doc（安装上boost库不能被muduo 使用，所以采用下载boost源码再安装）

boost安装细说：

> 最终使用的boost源码安装:(版本1.6.9：http://www.boost.org/users/history/version_1_69_0.html)
>
> 安装步骤：https://blog.csdn.net/zhangzq86/article/details/81082810
>
> 最终安装到了/usr/lcoal/boost目录下，所以需要添加环境变量(sudo vim /etc/profile)
>
> BOOST_INCLUDE_DIR=/usr/local/boost/include/boost
>
> export BOOST_INCLUDE_DIR
>
> BOOST_LIB=/usr/local/boost/lib
>
> export BOOST_LIB
>
> BOOST_ROOT=/usr/local/boost
>
> export BOOST_ROOT
>
> 之后(生效)：source /etc/profile
>
> 后：
>
> 出现错误：/usr/local/boost/include/boost/concept/detail/has_constraints.hpp:44:58: error: use of old-style cast [-Werror=old-style-cast]
> ​       , value = sizeof( detail::has_constraints_((Model*)0) ) == sizeof(detail::yes) );
>
> 解决办法：这是 boost 的 bug，需要把 CMakeLists.txt 中的 -Wold-style-cast 注释掉
>
> 出现问题：/usr/local/boost/include/boost/circular_buffer/base.hpp:1178:5: error: declaration of ‘alloc’ shadows a member of 'this' [-Werror=shadow]
> ​     : base(boost::empty_init_t(), alloc)
>
> 后出现问题无法解决：
>
> 所以决定将boost安装在默认目录下（版本：1.6.2）：这个会将头文件安装到/usr/local/include下，库文件安装到 /usr/local/lib
>
> ```html
> tar zxvf boost_1_62_0.tar.gz
> cd boost_1_62_0
> sudo ./bootstrap.sh
> sudo ./b2 install
> ```
>
> 还是没有装好，之后尝试在ubuntu下安装boost，再来安装muduo
>
> 最后没有在ubuntu下再试，之后在虚拟机Centos7Forceph**成功将muduo编译成功了**，重装了cmake,curl,c-ares DNS,google protobuf,然后boost采用的源码安装，版本为1.5.8,安装过程如上；
>
> 将muduo git clone下来之后，进行如下安装
>
> ```
> cd muduo
> ./build.sh (产生了release-cpp11文件夹，这个文件夹中的内容主要有，bin下的可执行文件和lib下的静态库文件)
> ./build.sh install (产生了release-install-cpp11文件夹，include下的muduo的头文件和lib下的静态库文件)
> ```
>
>

>

非必需：

a.curl : sudo yum install libcurl-devel

b.c-ares DNS： yum install c-ares-devel

c.Google Protobuf : https://github.com/protocolbuffers/protobuf/blob/master/src/README.md

# 第二部分 源码分析

## 2.1.Buffer类的分析

```
muduo EventLoop采用的是水平触发而不是边缘触发，原因如下：
1.兼容传统poll，因为在描述数较少，而活动描述符较多时，epoll不一定比poll更高效
2.水平编程更简单些，
3.读写的时候不必等候出现EAGAIN,可以减少系统调用次数
```

## 2.2.Buffer的读写

```
使用了readv和writev:可以发送一个结构体数组对应的数据，该结构体有两个数据，一个是一段连续数据的指针；第二个是该数据的长度
readv和writev函数的功能可以概括为：对数据进行整合传输以及发送。通过writev函数可以将分散保存在多个buff的数据一并进行发送，通过readv可以由多个buff分别接受数据，适当的使用这两个函数可以减少I/O函数的调用次数（readv和writev函数批量的进行数据的发送和读取）

在muduo中，它的iovec有两块，第一块指向muduo Buffer中的writeable字节，另一块指向栈上的extrabuf（65536字节）.如果数据不多，那么全部的数据就会读到Buffer中，如果数据长度超过Buffer的writable字节数，就会读到栈上的extrabuf,然后程序在把extrabuf里的数据append()到Buffer中（这样会导致Buffer的扩展）
这样做法的好处在于，利用了临时栈空间，避免每个连接初始的Buffer过大造成内存浪
```

```
Buffer不是线程安全的 
```

## 2.3.Buffer类的数据结构 

```
Buffer的内部是一个vector<char>,它是一块连续的内存
结构如下：
class Buffer{
    private:
    std::vector<char> buffer_;
    size_t readerIndex_;
    size_t writerIndex_;
};
另外两个指示位置的index，而不是char*是为了应对迭代器失效的情况

buffer的0位置到readerIndex_的位置为prependable部分
readerIndex_到writerIndex_的部分为readable部分
writerIndex_到buffer的末尾位置为writable部分

另外Buffer中还有两个常数，kCheapPrepend(8)和kInitialSize(1024),分别指示了prependable部分的大小（即第一次的读写buffer所在的位置），和writable的初始大小；即buffer的总长为8+1024=1032

在buffer操作注意的点：
1.readerIndex_和writerIndex会随着读写过程不断变化位置（但会维持readerIndex_<=writerIndex_）,但当他们某个时刻相等时，那么就会将他们指针移动到kCheapPrepend所指示的位置（回归原位置）
2.buffer利用vector的特性，自动增长
3.内部腾挪：在某些时候，经过若干次读写后，readIndex移到了比较靠后的位置，前面留下了比较大的prependable的空间，当后面又有比较大的写请求，超过了writable的返回，而前面的空间够，那么这个时候buffer就会选择将当前有效数据移动到kCheapprepend的位置，最后在写入数据

4.buffer的一个创新点：前方添加：提供了一个prependable空间，
```

