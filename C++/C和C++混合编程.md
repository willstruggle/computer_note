# C和C++混合编程(__cplusplus 与 external "c" 的使用)

C和C++混合编程(__cplusplus 与 external "c" 的使用)
第一种理解
比如说你用C++开发了一个DLL库，为了能够让C语言也能够调用你的DLL输出(Export)的函数，你需要用extern "C"来强制编译器不要修改你的
函数名。
通常，在C语言的头文件中经常可以看到类似下面这种形式的代码：
```
#ifdef __cplusplus
extern "C" {
#endif
/**** some declaration or so *****/
#ifdef __cplusplus
  }
#endif /* end of __cplusplus */
```
那么，这种写法什么用呢？实际上，这是为了让CPP能够与C接口而采用的一种语法形式。之所以采用这种方式，是因为两种语言之间的一些差
异所导致的。由于CPP支持多态性，也就是具有相同函数名的函数可以完成不同的功能，CPP通常是通过参数区分具体调用的是哪一个函数。在
编译的时候，CPP编译器会将参数类型和函数名连接在一起，于是在程序编译成为目标文件以后，CPP编译器可以直接根据目标文件中的符号名
将多个目标文件连接成一个目标文件或者可执行文件。但是在C语言中，由于完全没有多态性的概念，C编译器在编译时除了会在函数名前面添
加一个下划线之外，什么也不会做（至少很多编译器都是这样干的）。由于这种的原因，当采用CPP与C混合编程的时候，就可能会出问题。假
设在某一个头文件中定义了这样一个函数：
```
int foo(int a, int b);
```
而这个函数的实现位于一个.c文件中，同时，在.cpp文件中调用了这个函数。那么，当CPP编译器编译这个函数的时候，就有可能会把这个函数
名改成_fooii，这里的ii表示函数的第一参数和第二参数都是整型。而C编译器却有可能将这个函数名编译成_foo。也就是说，在CPP编译器得
到的目标文件中，foo()函数是由_fooii符号来引用的，而在C编译器生成的目标文件中，foo()函数是由_foo指代的。但连接器工作的时候，它
可不管上层采用的是什么语言，它只认目标文件中的符号。于是，连接器将会发现在.cpp中调用了foo()函数，但是在其它的目标文件中却找不
到_fooii这个符号，于是提示连接过程出错。extern "C" {}这种语法形式就是用来解决这个问题的。本文将以示例对这个问题进行说明。
首先假设有下面这样三个文件：
```
/* file: test_extern_c.h */
#ifndef __TEST_EXTERN_C_H__
#define __TEST_EXTERN_C_H__
#ifdef __cplusplus
extern "C" {
#endif
/*
* this is a test function, which calculate
* the multiply of a and b.
*/
extern int ThisIsTest(int a, int b);
#ifdef __cplusplus
  }
#endif /* end of __cplusplus */
#endif
```
在这个头文件中只定义了一个函数，ThisIsTest()。这个函数被定义为一个外部函数，可以被包括到其它程序文件中。假设ThisIsTest()函数
的实现位于test_extern_c.c文件中：
```
/* test_extern_c.c */
#include "test_extern_c.h"
int ThisIsTest(int a, int b)
{
  return (a + b);
}
```
可以看到，ThisIsTest()函数的实现非常简单，就是将两个参数的相加结果返回而已。现在，假设要从CPP中调用ThisIsTest()函数：
```
/* main.cpp */
#include "test_extern_c.h"
#include <stdio.h>
#include <stdlib.h>
class FOO {
  public:
  int bar(int a, int b)
    {
        printf("result=%i/n", ThisIsTest(a, b));
    }
};
int main(int argc, char **argv)
{
  int a = atoi(argv[1]);
  int b = atoi(argv[2]);
  FOO *foo = new FOO();
  foo->bar(a, b);
  return(0);
}
```
在这个CPP源文件中，定义了一个简单的类FOO，在其成员函数bar()中调用了ThisIsTest()函数。下面看一下如果采用gcc编译test_extern_c.c
，而采用g++编译main.cpp并与test_extern_c.o连接会发生什么情况：
```
[cyc@cyc src]$ gcc -c test_extern_c.c
[cyc@cyc src]$ g++ main.cpp test_extern_c.o
[cyc@cyc src]$ ./a.out 4 5          
result=9
```
可以看到，程序没有任何异常，完全按照预期的方式工作。那么，如果将test_extern_c.h中的extern "C" {}所在的那几行注释掉会怎样呢？
注释后的test_extern_c.h文件内容如下：
```
/* test_extern_c.h */
#ifndef __TEST_EXTERN_C_H__
#define __TEST_EXTERN_C_H__
//#ifdef   __cplusplus
//extern "C" {
//#endif
/*
/* this is a test function, which calculate
* the multiply of a and b.
*/
extern int ThisIsTest(int a, int b);
//#ifdef   __cplusplus
// }
//#endif   /* end of __cplusplus */
#endif
```
之外，其它文件不做任何的改变，仍然采用同样的方式编译test_extern_c.c和main.cpp文件：
```
[cyc@cyc src]$ gcc -c test_extern_c.c
[cyc@cyc src]$ g++ main.cpp test_extern_c.o
/tmp/cca4EtJJ.o(.gnu.linkonce.t._ZN3FOO3barEii+0x10): In function `FOO::bar(int, int)':
: undefined reference to `ThisIsTest(int, int)'
collect2: ld returned 1 exit status
```
在编译main.cpp的时候就会出错，连接器ld提示找不到对函数ThisIsTest()的引用。
为了更清楚地说明问题的原因，我们采用下面的方式先把目标文件编译出来，然后看目标文件中到底都有些什么符号：
```
[cyc@cyc src]$ gcc -c test_extern_c.c  
[cyc@cyc src]$ objdump -t test_extern_c.o
test_extern_c.o:   file format elf32-i386
SYMBOL TABLE:
00000000 l   df *ABS* 00000000 test_extern_c.c
00000000 l   d .text 00000000
00000000 l   d .data 00000000
00000000 l   d .bss   00000000
00000000 l   d .comment     00000000
00000000 g   F .text 0000000b ThisIsTest
[cyc@cyc src]$ g++ -c main.cpp      
[cyc@cyc src]$ objdump -t main.o      
main.o:   file format elf32-i386
MYMBOL TABLE:
00000000 l   df *ABS* 00000000 main.cpp
00000000 l   d .text 00000000
00000000 l   d .data 00000000
00000000 l   d .bss   00000000
00000000 l   d .rodata     00000000
00000000 l   d .gnu.linkonce.t._ZN3FOO3barEii 00000000
00000000 l   d .eh_frame     00000000
00000000 l   d .comment     00000000
00000000 g   F .text 00000081 main
00000000       *UND* 00000000 atoi
00000000       *UND* 00000000 _Znwj
00000000       *UND* 00000000 _ZdlPv
00000000 w   F .gnu.linkonce.t._ZN3FOO3barEii 00000027 _ZN3FOO3barEii
00000000       *UND* 00000000 _Z10ThisIsTestii
00000000       *UND* 00000000 printf
00000000       *UND* 00000000 __gxx_personality_v0
```
可以看到，采用gcc编译了test_extern_c.c之后，在其目标文件test_extern_c.o中的有一个ThisIsTest符号，这个符号就是源文件中定义的
ThisIsTest()函数了。而在采用g++编译了main.cpp之后，在其目标文件main.o中有一个_Z10ThisIsTestii符号，这个就是经过g++编译器“粉
碎”过后的函数名。其最后的两个字符i就表示第一参数和第二参数都是整型。而为什么要加一个前缀_Z10我并不清楚，但这里并不影响我们的
讨论，因此不去管它。显然，这就是原因的所在，其原理在本文开头已作了说明。
那么，为什么采用了extern "C" {}形式就不会有这个问题呢，我们就来看一下当test_extern_c.h采用extern "C" {}的形式时编译出来的目标
文件中又有哪些符号：
```
[cyc@cyc src]$ gcc -c test_extern_c.c
[cyc@cyc src]$ objdump -t test_extern_c.o
test_extern_c.o:   file format elf32-i386
SYMBOL TABLE:
00000000 l   df *ABS* 00000000 test_extern_c.c
00000000 l   d .text 00000000
00000000 l   d .data 00000000
00000000 l   d .bss   00000000
00000000 l   d .comment     00000000
00000000 g   F .text 0000000b ThisIsTest
[cyc@cyc src]$ g++ -c main.cpp
[cyc@cyc src]$ objdump -t main.o
main.o:   file format elf32-i386
SYMBOL TABLE:
00000000 l   df *ABS* 00000000 main.cpp
00000000 l   d .text 00000000
00000000 l   d .data 00000000
00000000 l   d .bss   00000000
00000000 l   d .rodata     00000000
00000000 l   d .gnu.linkonce.t._ZN3FOO3barEii 00000000
00000000 l   d .eh_frame     00000000
00000000 l   d .comment     00000000
00000000 g   F .text 00000081 main
00000000       *UND* 00000000 atoi
00000000       *UND* 00000000 _Znwj
00000000       *UND* 00000000 _ZdlPv
00000000 w   F .gnu.linkonce.t._ZN3FOO3barEii 00000027 _ZN3FOO3barEii
00000000       *UND* 00000000 ThisIsTest
00000000       *UND* 00000000 printf
00000000       *UND* 00000000 __gxx_personality_v0
```
注意到这里和前面有什么不同没有，可以看到，在两个目标文件中，都有一个符号ThisIsTest，这个符号引用的就是ThisIsTest()函数了。显然，此时在两个目标文件中都存在同样的ThisIsTest符号，因此认为它们引用的实际上同一个函数，于是就将两个目标文件连接在一起，凡是出现程序代码段中有ThisIsTest符号的地方都用ThisIsTest()函数的实际地址代替。另外，还可以看到，仅仅被extern "C" {}包围起来的函数采用这样的目标符号形式，对于main.cpp中的FOO类的成员函数，在两种编译方式后的符号名都是经过“粉碎”了的。因此，综合上面的分析，我们可以得出如下结论：采用extern "C" {} 这种形式的声明，可以使得CPP与C之间的接口具有互通性，不会由于语言内部的机制导致连接目标文件的时候出现错误。需要说明的是，上面只是根据我的试验结果而得出的结论。由于对于CPP用得不是很多，了解得也很少，因此对其内部处理机制并不是很清楚，如果需要深入了解这个问题的细节请参考相关资料。
 
第二种理解
时常在cpp的代码之中看到这样的代码:
```
#ifdef __cplusplus
extern "C" {
#endif

//一段代码

#ifdef __cplusplus
}
#endif
 ```
　　这样的代码到底是什么意思呢？首先，__cplusplus是cpp中的自定义宏，那么定义了这个宏的话表示这是一段cpp的代码，也就是说，上面的代码的含义是:如果这是一段cpp的代码，那么加入extern "C"{和}处理其中的代码。要明白为何使用extern "C"，还得从cpp中对函数的重载处理开始说起。在c++中，为了支持重载机制，在编译生成的汇编码中，要对函数的名字进行一些处理，加入比如函数的返回类型等等.而在C中，只是简单的函数名字而已，不会加入其他的信息.也就说:C++和C对产生的函数名字的处理是不一样的.比如下面的一段简单的函数，我们看看加入和不加入extern "C"产生的汇编代码都有哪些变化:
```
int f(void)
{
return 1;
}
```
　　在加入extern "C"的时候产生的汇编代码是:
```
.file "test.cxx"
.text
.align 2
.globl _f
.def _f; .scl 2; .type 32; .endef
_f:
pushl %ebp
movl %esp， %ebp
movl $1， %eax
popl %ebp
ret
```
　　但是不加入了extern "C"之后
```
.file "test.cxx"
.text
.align 2
.globl __Z1fv
.def __Z1fv; .scl 2; .type 32; .endef
__Z1fv:
pushl %ebp
movl %esp， %ebp
movl $1， %eax
popl %ebp
ret
``` 
　　两段汇编代码同样都是使用gcc -S命令产生的，所有的地方都是一样的，唯独是产生的函数名，一个是_f，一个是__Z1fv。明白了加入与不加入extern "C"之后对函数名称产生的影响，我们继续我们的讨论:为什么需要使用extern "C"呢？C++之父在设计C++之时，考虑到当时已经存在了大量的C代码，为了支持原来的C代码和已经写好C库，需要在C++中尽可能的支持C，而extern "C"就是其中的一个策略。试想这样的情况:一个库文件已经用C写好了而且运行得很良好，这个时候我们需要使用这个库文件，但是我们需要使用C++来写这个新的代码。如果这个代码使用的是C++的方式链接这个C库文件的话，那么就会出现链接错误.我们来看一段代码:首先，我们使用C的处理方式来写一个函数，也就是说假设这个函数当时是用C写成的:
```
//f1.c
extern "C"
{
void f1()
{
return;
}
}
 ```
　　编译命令是:gcc -c f1.c -o f1.o 产生了一个叫f1.o的库文件。再写一段代码调用这个f1函数:
```
// test.cxx
//这个extern表示f1函数在别的地方定义，这样可以通过
//编译，但是链接的时候还是需要
//链接上原来的库文件.
extern void f1();
 
int main()
{
f1();
 
return 0;
}
```
　　通过gcc -c test.cxx -o test.o 产生一个叫test.o的文件。然后，我们使用gcc test.o f1.o来链接两个文件，可是出错了，错误的提示
是:
```
test.o(.text + 0x1f):test.cxx: undefine reference to 'f1()'
```
　　也就是说，在编译test.cxx的时候编译器是使用C++的方式来处理f1()函数的，但是实际上链接的库文件却是用C的方式来处理函数的，所以就会出现链接过不去的错误:因为链接器找不到函数。因此，为了在C++代码中调用用C写成的库文件，就需要用extern "C"来告诉编译器:这是一个用C写成的库文件，请用C的方式来链接它们。比如，现在我们有了一个C库文件，它的头文件是f.h，产生的lib文件是f.lib，那么我们如果要在C++中使用这个库文件，我们需要这样写:
```
extern "C"
{
#include "f.h"
}
```
　　回到上面的问题，如果要改正链接错误，我们需要这样子改写test.cxx:
```
extern "C"
{
extern void f1();
}
 
int main()
{
f1();
 
return 0;
}
```
重新编译并且链接就可以过去了.

总结
C和C++对函数的处理方式是不同的.extern "C"是使C++能够调用C写作的库文件的一个手段，如果要对编译器提示使用C的方式来处理函数的话，那么就要使用extern "C"来说明。