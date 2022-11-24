# GDB调试工具

GDB是GNU开源组织发布的一个强大的UNIX下调试程序工具。或许各位比较喜欢那种图形界面方式的，像VC，BCB等IDE的调试，但如果你是在UNIX平台下作软件，你会发现GDB这个调试工具有比VC，BCB的图形化调试器更强大的功能。所谓“寸有所长，尺有所短”就是这个道理。
 
一般来说，GDB主要帮助你完成下面四个方面的功能：
1、启动你的程序，可以按照你自定义的要求随心所欲的运行程序。
2、可以让调试程序在你所指定的位置的断点处停止。
3、当程序停止时，可以检查此时你的程序中所发生的事情。
4、动态的改变你程序的执行环境。
  从上面看来，GDB和一般的调试工具没有什么两样，基本上也是完成这些功能，不过在细节上，你会发现GDB这个调试工具的强大，大家可能习惯图形化的调试工具，但有时候，命令行的调试工具却有着图形化工具所不能完成的功能。
 
让我们来看一个例子：
  源程序：test.c
```
    1 #include <stdio.h>
    2
    3 int func(int n)
    4 {
    5         int sum=0,i;
    6         for(i=0; i<n; i++)
    7         {
    8                 sum+=i;
    9         }
   10         return sum;
   11 }
   12
   13
   14 main()
   15 {
   16         int i;
   17         long result = 0;
   18         for(i=1; i<=100; i++)
   19         {
   20                 result += i;
   21         }
   22
   23        printf("result[1-100] = %d /n", result );
   24        printf("result[1-250] = %d /n", func(250) );
   25 }
```
编译生成执行文件：
    <byb>gcc –g test.c –o test
 
使用GDB调试：
  <byb>gdb test    ——————启动GDB
```
  GNU gdb 5.1.1
Copyright 2002 Free Software Foundation, Inc.
GDB is free software, covered by the GNU General Public License, and you are
welcome to change it and/or distribute copies of it under certain conditions.
Type "show copying" to see the conditions.
There is absolutely no warranty for GDB. Type "show warranty" for details.
This GDB was configured as "i386-suse-linux"...
  (gdb) list ——————从第一行列出源码 
  1        #include <stdio.h>
2
3        int func(int n)
4        {
5                int sum=0,i;
6                for(i=0; i<n; i++)
7                {
8                        sum+=i;
9                }
10             return sum;
  (gdb)    ——————直接回车表示，重复上一次命令
  11       }
12
13
14       main()
15       {
16               int i;
17               long result = 0;
18               for(i=1; i<=100; i++)
19               {
20                       result += i;   
  (gdb) break 16  ——————设置断点，在源程序第16行处。
  Breakpoint 1 at 0x8048496: file tst.c, line 16.
  (gdb) break func ——————设置断点，在函数func()入口处。
  Breakpoint 2 at 0x8048456: file tst.c, line 5.
  (gdb) info break  ——————查看断点信息。
  Num Type           Disp Enb Address    What
1   breakpoint     keep y   0x08048496 in main at tst.c:16
2   breakpoint     keep y   0x08048456 in func at tst.c:5
  (gdb) run        ——————运行程序
Starting program: /home/hchen/test/tst
Breakpoint 1, main () at tst.c:17    ——————在断点处停住。
17               long result = 0;
  (gdb) next        ——————单条语句执行。
  18               for(i=1; i<=100; i++)
  (gdb) n
20                       result += i;
(gdb) n
18               for(i=1; i<=100; i++)
(gdb) n
20                       result += i;
  (gdb) continue   ——————继续运行程序
Continuing.
result[1-100] = 5050      ——————程序输出。
Breakpoint 2, func (n=250) at tst.c:5
5                int sum=0,i;
(gdb) n
6                for(i=1; i<=n; i++)
  (gdb) print i   ——————打印变量i的值。
  $1 = 134513808
(gdb) n
8                        sum+=i;
(gdb) n
6                for(i=1; i<=n; i++)
(gdb) p sum
$2 = 1
(gdb) n
8                        sum+=i;
(gdb) p i
$3 = 2
(gdb) n
6                for(i=1; i<=n; i++)
(gdb) p sum    ——————p是print的缩写
$4 = 3
  (gdb) bt    ——————查看函数堆栈
  #0 func (n=250) at tst.c:5
#1 0x080484e4 in main () at tst.c:24
#2 0x400409ed in __libc_start_main () from /lib/libc.so.6
  (gdb) finish    ——————推出函数
  Run till exit from #0 func (n=250) at tst.c:5
0x080484e4 in main () at tst.c:24
24              printf("result[1-250] = %d /n", func(250) );
Value returned is $6 = 31375
  (gdb) continue
Continuing.
result[1-250] = 31375   ——————程序输出。
Program exited with code 027.——————程序退出，调试结束。
(gdb) quit     ——————退出gdb
```
  好了，有了以上的感性认识，还是让我们来系统的认识一下gdb吧。
 
使用GDB
 
一般来说GDB主要调试的是C/C++程序。要调试C/C++程序，首先在编译时，我们必须要把调试信息加到可执行文件中。使用编译器（cc/gcc/g++）的-g参数可以做到这一点，如：
```
$ cc –g hello.c –o hello
$ g++ -g hello.cpp –o hello
```
如果没有-g，你将看不见程序的函数名，变量名，所代替的全是运行的内存地址。当你用-g把调试信息假如之后，并成功编译目标代码以后，让我们来看看如果用GDB调试它。
  启动GDB的方法有以下几种：
1、  gdb <program>
program也就是你的执行文件，一般在当前目录下。
2、  gdb <program> core
用gdb同时调试一个运行程序和core文件，core是程序非法执行后core dump后产生的文件。
3、  gdb <program> <PID>
如果你的程序是一个服务程序，那么你可以指定这个服务程序运行时的进程ID。gdb会自动attach上去，并调试它。program应该在PATH环境变量中搜索到。
 
  GDB启动时，可以加上一些GDB的启动开关，详细的开关可以用gdb –help来查看。下面只列举一些比较常用的参数：
-symbols <file>
-s <file>
从指定文件中读取符号表。
 
-se file
从指定文件中读取符号表信息，并把他用在可执行文件中。
-core <file>
-c <file>
调试core dump的core文件。
 
-directory <directory>
-d <directory>
加入一个源文件的搜索路径。默认搜索路径是环境变量中PATH所定义的路径。
 
 
GDB的命令概貌
  启动gdb后，就进入了gdb的调试环境，就可以使用gdb的命令开始调试程序了，gdb的命令可以使用help命令来查看，如下所示：
```
(gdb) help
List of classes of commands:
aliases -- Aliases of other commands
breakpoints -- Making program stop at certain points
data -- Examining data
files -- Specifying and examining files
internals -- Maintenance commands
obscure -- Obscure features
running -- Running the program
stack -- Examining the stack
status -- Status inquiries
support -- Support facilities
tracepoints -- Tracing of program execution without stopping the program
user-defined -- User-defined commands
 
Type "help" followed by a class name for a list of commands in that class.
Type "help" followed by command name for full documentation.
Command name abbreviations are allowed if unambiguous.
(gdb)
```
gdb的命令很多，gdb把之分成很多种类。help命令只是列出了gdb的命令种类，如果要看种类中的命令，使用help <class>命令，如：help breakpoints，查看设置断点的所有命令。也可以直接help <command>来查看命令的帮助。
Gdb中，输入命令时，可以不用打全命令，只用打命令的前几个字符就可以了，当然，命令的前几个字符要标志着一个唯一的命令，在linux下，可以敲击两次TAB键来补齐命令的全称，如果有重复的，gdb会把其列出来。
 
示例一：在进入函数func时，设置一个断点。可以敲击break func，或者直接就是b func
```
(gdb) b func
Breakpoint 1 at 0x8048458: file hello.c, line 10.
```
示例二：敲入b按两次TAB键，你会看到所有b开头的命令：
```
(gdb) b
backtrace break bt
```
示例三：只记得函数的前缀，可以这样：
```
(gdb) b make_<按TAB键>
make_a_section_from_file     make_environ
make_abs_section             make_function_type
make_blockvector             make_pointer_type
make_cleanup                 make_reference_type
make_command                 make_symbol_completion_list
(gdb) b make_
```
GDB把所有make开头的函数全部列出来给你查看。
   
示例四：调试C++程序，可以函数名一样。如：
```
(gdb) b ‘bubble(<按两次TAB键>
bubble(double,double)    bubble(int,int)
(gdb) b ‘bubble
```
你可以查看到C++中所有的重载函数以及参数
 
  要退出GDB，只要quit或命令简称q就行了。