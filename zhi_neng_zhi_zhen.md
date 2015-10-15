# 智能指针

###2.1. ﻿概述

1998年修订的第一版C++标准只提供了一种智能指针： std::auto_ptr 。 它基本上就像是个普通的指针： 通过地址来访问一个动态分配的对象。 std::auto_ptr 之所以被看作是智能指针，是因为它会在析构的时候调用 delete 操作符来自动释放所包含的对象。 当然这要求在初始化的时候，传给它一个由 new 操作符返回的对象的地址。 既然 std::auto_ptr 的析构函数会调用 delete 操作符，它所包含的对象的内存会确保释放掉。 这是智能指针的一个优点。

当和异常联系起来时这就更加重要了：没有 std::auto_ptr 这样的智能指针，每一个动态分配内存的函数都需要捕捉所有可能的异常，以确保在异常传递给函数的调用者之前将内存释放掉。 Boost C++ 库 Smart Pointers 提供了许多可以用在各种场合的智能指针。

###2.2. RAII
智能指针的原理基于一个常见的习语叫做 RAII ：资源申请即初始化。 智能指针只是这个习语的其中一例——当然是相当重要的一例。 智能指针确保在任何情况下，动态分配的内存都能得到正确释放，从而将开发人员从这项任务中解放了出来。 这包括程序因为异常而中断，原本用于释放内存的代码被跳过的场景。 用一个动态分配的对象的地址来初始化智能指针，在析构的时候释放内存，就确保了这一点。 因为析构函数总是会被执行的，这样所包含的内存也将总是会被释放。

无论何时，一定得有第二条指令来释放之前另一条指令所分配的资源时，RAII 都是适用的。 许多的 C++ 应用程序都需要动态管理内存，因而智能指针是一种很重要的 RAII 类型。 不过 RAII 本身是适用于许多其它场景的。

```#include <windows.h> 

class windows_handle 
{ 
  public: 
    windows_handle(HANDLE h) 
      : handle_(h) 
    { 
    } 

    ~windows_handle() 
    { 
      CloseHandle(handle_); 
    } 

    HANDLE handle() const 
    { 
      return handle_; 
    } 

  private: 
    HANDLE handle_; 
}; 

int main() 
{ 
  windows_handle h(OpenProcess(PROCESS_SET_INFORMATION, FALSE, GetCurrentProcessId())); 
  SetPriorityClass(h.handle(), HIGH_PRIORITY_CLASS); 
} ```
下载源代码
上面的例子中定义了一个名为 windows_handle 的类，它的析构函数调用了 CloseHandle() 函数。 这是一个 Windows API 函数，因而这个程序只能在 Windows 上运行。 在 Windows 上，许多资源在使用之前都要求打开。 这暗示着一旦资源不再使用之后就应该关闭。 windows_handle 类的机制能确保这一点。

windows_handle 类的实例以一个句柄来初始化。 Windows 使用句柄来唯一的标识资源。 比如说，OpenProcess() 函数返回一个 HANDLE 类型的句柄，通过该句柄可以访问当前系统中的进程。 在示例代码中，访问的是进程自己——换句话说就是应用程序本身。

我们通过这个返回的句柄提升了进程的优先级，这样它就能从调度器那里获得更多的 CPU 时间。 这里只是用于演示目的，并没什么实际的效应。 重要的一点是：通过 OpenProcess() 打开的资源不需要显示的调用 CloseHandle() 来关闭。 当然，应用程序终止时资源也会随之关闭。 然而，在更加复杂的应用程序里， windows_handle 类确保当一个资源不再使用时就能正确的关闭。 某个资源一旦离开了它的作用域——上例中 h 的作用域在 main() 函数的末尾——它的析构函数会被自动的调用，相应的资源也就释放掉了。