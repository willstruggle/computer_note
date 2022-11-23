# call_once
   在多线程的环境下，有些时候我们不需要某给函数被调用多次或者某些变量被初始化多次，它们仅仅只需要被调用一次或者初始化一次即可。很多时候我们为了初始化某些数据会写出如下代码，这些代码在单线程中是没有任何问题的，但是在多线程中就会出现不可预知的问题。
```
bool initialized = false; // global flag
if (!initialized) {
    // initialize if not initialized yet
    initialize ();
    initialized = true;
}
or
static std::vector<std::string> staticData;
void foo ()
{
    if (staticData.empty ()) {
        staticData = initializeStaticData ();
    }
    ...
}
```
为了解决上述多线程中出现的资源竞争导致的数据不一致问题，我们大多数的处理方法就是使用互斥锁来处理。在C++11中提供了最新的处理方法：使用std::call_once函数来处理，其定义如下头文件`#include<mutex>`
```
template< class Function, class... Args >
void call_once ( std::once_flag& flag, Function&& f, Args&& args... );
```
参数解析：
flag     -	 an object, for which exactly one function gets executed
f	 -	 需要被调用的函数
args...  -	 传递给函数f的参数（可以多个）
返回值为 (none)
抛出异常
```
std::system_error if any condition prevents calls to call_once from executing as specified any exception thrown by f
```
例:
```
static std::vector<std::string> staticData;
std::vector<std::string>initializeStaticData ()
{
    std::vector<std::string> vec;
    vec.push_back ("initialize");
 
    return vec;
}
 
void foo()
{
    static std::once_flag oc;
    std::call_once(oc, [] { staticData = initializeStaticData ();});
}
```
正如上面的例子所示call_once函数第一个参数是std::once_flag的一个对象，第二个参数可以是函数、成员函数、函数对象、lambda函数。

2、实例1：
```
#include <iostream>
#include <thread>
#include <mutex>
 
std::once_flag flag1;
 
void simple_do_once()
{
    std::call_once(flag1, [](){ std::cout << "Simple example: called once\n"; });
}
 
int main()
{
    std::thread st1(simple_do_once);
    std::thread st2(simple_do_once);
    std::thread st3(simple_do_once);
    std::thread st4(simple_do_once);
    st1.join();
    st2.join();
    st3.join();
    st4.join();
}
```

Simple example: called once
实例2：单例模式：
```
some_class *the_instance;
std::once_flag instance_created;
some_class *get_instance() {
    std::call_once(
        instance_created,
        [](){ the_instance=new some_class(...); }
    );
    return the_instance;
}
 ```