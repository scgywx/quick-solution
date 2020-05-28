# A Word Solution
Solve problem in a word

## Linux
+ tcpdump或者wireshark抓包发现数据包大于mtu，是tso的作用

+ php的mysqlnd扩展connect超时使用default_socket_timeout，而read则是使用mysqlnd.read_timeout【默认是1年】, 所以如果想设置mysql超时需要两个配置一起修改（注：mysqlnd.read_timeout只能改php.ini）

+ socket关闭时，通常不能完整发送在缓冲区的数据，如果想发送还在缓冲区的数据，需要设置LINGER。

+ 信号量（semaphore）、互斥（mutex）、临界区（critical section）的区别，  
	信号量可以唤醒N个线程，并且任意线程都可以释放（-1操作），  
	互斥只能唤醒一个线程，并且遵循谁获取谁释放的原则，  
	临界区与前两个的区别是，临界区不能跨进程，前两者则可以。  

+ 多线程编程了解一下volatile和barrier防止编译优化与cpu乱序执行--《程序员的自我修养》

+ linux的fork执行很快，并且不需要copy内存，COW了解一下。

+ 如果socket数据没读完，另一端关闭连接，使用recv的MSG_PEEK将不会得到错误。

+ 程序自修改(Self-Modifying Code)，不能将代码放在.text段，因为.text段所在内存是只读，不可以在运行时修改。解决方法是把代码放到.data段，.data段的内存不仅可以执行还可以修改，这样就可以在运行时修改代码了（一些绕病毒检测工具的程序就这么干）

+ systemtap分析CPU、内存、IO、网络等系统级相关问题的工具，非常强大。

+ openssl命令行的key和iv需要密钥的16进制，程序中使用字符串即可

## PHP
+ PHP DES算法需要添加padding，否则其它语言解不出来，反解不需要unpadding.

+ PHP进程出现卡住，如果cpu持续偏高，说明死循环，检查while的使用和for的变量复用。

+ phpredis3.2版，对cluster会出现Error processing EXEC across the cluster错误，3.0版本无此问题

+ 大量使用file_get_contents将会出现连接不上服务器，改用curl即可，还不知道为何

+ 夏令时会导致某一天不足24小时(即不足86400秒)，所以不能直接使用+86400来达到增加一天.的目的，可以使用strtotime(date('Y-m-d H:i:s') . ' +1 day');

+ 高并发的应用，不要把FPM开得太多，否则会出现大规模的timeout.

+ php连接mysql出现No such file or directory，改成ip或者设置unixdomain地址.

+ yaf参数控制bug会导致进程崩溃，https://github.com/laruence/yaf/issues/420

+ php的protobuf扩展(非官方)，使用空的结构体会报错(嵌套空结构体)，修改protobuf.c的serializeToString，让其返回空串，而不是出错。

+ 34、yaf会自动将controller转换为小写，修改yaf_dispatcher.c的yaf_dispatcher_fix_default函数  
char *q, *p = zend_str_tolower_dup(Z_STRVAL_P(controller), Z_STRLEN_P(controller));  
改成  
char *q, *p = estrndup(Z_STRVAL_P(controller), Z_STRLEN_P(controller));  

+ 查看php当前执行的opcode  
(gdb) p *executor_globals.current_execute_data.opline

+ md5('240610708') == md5('QNKCDZO') 返回为真  
PHP中所有以0e开头并且后续全是数字的字符串比较，一律返回true.  
因为PHP的类型转换认为这是数字的科学记数法表示。  

+ 在使用iconv从utf-8转到gbk时出错了，因为遇到了\c2\a0字符，这是个特殊空格，可以使用//IGNORE忽略，也可以在转换之前替换掉

+ redis在局域网与本机通信存在网络开销，内网延迟0.3ms(平均3条命令就1ms），本机延迟0.03ms(30条命令才1ms)，10倍差距

## Golang
+ reflect的set必须要指针类型，ValueOf的时候传递指针类型

+ go的gc不会将内存归还给os。

+ go build -gcflags "-l -N"  
-l   关掉内联  
-N  关掉优化  

## C/C++
+ epoll的ET模式下有两点要特别注意：  
一、在epoll_wait之后必须处理read和write，不能只处理read或者只处理write，因为一旦使用了ET模式，而此时read和write事件同时触发，那么将有一个事件永远没机会处理。   
二、write与read必须要等到返回EAGAIN，否则注册事件不会触发.

+ phpredis扩展在长连接下的multi会出现协议错误
所以在multi之前，要先调用一下multi(2)，再调用discard。因为multi方法只有在pipeline模式下会清理callbacklist。如果是长连接还使用了multi的就一定要注意了。(3x版本以下都有此问题)

+ C语言三目运算简写  
    a ? : b  的效果跟  a ? a : b  一样

+ 宏定义返回值  
	```#define A(v)   ({ aaa; bbb ;ccc})  //这行代码为返回ccc表达式的值```

+ gcc的-l参数最好写在最后，否则连接时可能找不到符号(具体见-l参数详解)，一般参数顺序为：基本参数(如：-o -I -L等)、连接文件(如: *.a *.o)、库文件

+ 使用ssl库提示找不到符号：OPENSSL_add_all_algorithms_noconf，添加-lcrypto即可

+ c语言在64位下参数如果小于7个，分别从左向右放入rdi, rsi, rdx, rcx, r8, r9寄存器，大于7个，前6个方式相同，剩下的从右向左压入栈。相关知识点：x64 call convention

+ 已初始化的全局变量(非0）放在.data段，即需要占用可执行文件空间。未初始化的全局变量(默认值或者没有显示指明值）放在.bss段，不占用可执行文件大小，仅标记变量内存大小。

+ C++传递数组引用不会退化为指针，如:  test(int (&arr)[1024])，使用sizeof(arr)可以获取正确的内存大小。

+ C++的纯虚函数可以有实现(之前一直以为只能声明，不能定义)，leveldb的源代码中看到的。

+ gcc的__builtin_expect可以帮助CPU处理分支预测能力，优化后的代码更紧凑，减少跳转，提升性能，linux内核代码使用相当频繁(likely和unlikely)。

+ C++不兼容C部分  
a) sizeof('a')，   C是4,  C++是1  
b) enum转换，  C可以将一个enum类型转换为另一个enum类型，C++会报错  
其它网上说的什么结构体啊、变量声明顺序啊，都不能叫不兼容，毕竟是超集，比原来更好用这叫新特性，不兼容是指完全不能使用，或者说使用后出现不一样的结果。 

+ gdb调试常用  
layout src //显示源代码界面  
layout asm //显示汇编界面  
layout split //同时显示源代码和汇编界面    
disass /m 函数名  
set disassemble-next-line on //设置自动显示每行代码的反汇编  
si/ni //汇编级别单步  
set print address on  参数地址打印  
x/nfu 显示内存数据  
x 是 examine 的缩写  
n表示要显示的内存单元的个数  
f表示显示方式, 可取如下值    
u表示一个地址单元的长度  
例：x/8xw  显示8组4字节的内存数据，每组用16进制显示  

+ gcc指定库名
-Wl,-soname,libxxx.so.0  //指定库名  
-Wl,--rpath  //设置运行时库路径  
-Wl,--rpath-link //设置连接时库路径  

+ C/C++的undefine behavior行为  
```cpp
#include <stdlib.h>
#include <stdio.h>
int *ptr;
int index(){
    ptr = (int*)malloc(sizeof(int));
    ptr[0] = 2;
    return 0;
}
int main(){
    ptr = (int*)malloc(sizeof(int));
    ptr[0] = 1;
    printf("%d\n", ptr[index()]);
    return 0;
}
```  

+ c/c++循环依赖结构体或者类，需要将其中一个类依赖的类型改成指针，并且使用typedef struct或者class进行预申明，g++ -E可以很明确看出问题.

+ gcc使用-g和-pg开启性能检测  
./test   执行之后会生成gmon.out文件  
gprof ./test gmon.out |gprof2dot|dot -Tpng -o test.png 生成调用关系和消耗图  

+ 多线程使用条件变量时，一定要将条件放在wait之前，否则可能永远没机会执行了

+ c++20新特性，允许auto作为函数形参类型(template语法糖),  
专业术语：Abbreviated function templates

+ gcc playground: https://godbolt.org/

+ c++的符号重组术语：demangle

+ 导出指定函数代码  
objdump -d $FILENAME | sed '/<$FUNCTION>:/,/^$/!d'

+ 导出指定符号内容  
objdump -s --start-address=0xxxx--stop-address=0xxxx $FILENAME

+ C++ type_traits  
is_base_of 判断是否是继承类(编译器内部使用，非函数)  
enable_if 检测编译条件是否成立(编译器内部使用，非函数)  
如下代码可以限制模块类参数必须是Anim的子类：  
```
template <class T, typename std::enable_if<std::is_base_of<Anim, T>::value, int>::type = 0>
class Factory{};
```

+ gcc静态编译多线程库时，需要使用-pthread，而不能使用-lpthread，因为可能涉及到一些安全问题，前者会新增_REENTRANT宏，后者不会。

+ 火焰图生成  
git clone https://github.com/brendangregg/FlameGraph.git   
perf record -F 99 -g -p 31612 -- sleep 30   采样  
perf report -n 查看报告  
perf script | ./stackcollapse-perf.pl | ./flamegraph.pl > perf.svg  
注解：  
perf script    生成折叠后的调用栈  
./stackcollapse-perf.pl  生成火焰图  
./flamegraph.pl > perf.svg   生成svg  

+ memcpy如果src和dst有可能重叠（无论是向前还是向后），一定要换成memmove，否则copy数据会出错，因为有些库在memcpy的时候是从后往前copy的。

+ c++的copy elision特性  
消除了重复的构造函数调用，比如Test a(Test())，将只会调用一次构造函数  
gcc可以添加-fno-elide-constructors参数跳过此特性  

+ for(auto i : arr) 与 for(auto i = arr.begin(); i != arr.end(); i++)的区别  
前者end是在循环之前就调用的  
后者end是每次循环都调用的  

+ [&]{}与[=]()mutable{}的区别  
前者可以修改捕获的变量，并且影响原值  
后者可以修改捕获的变量，但是不影响原值，只影响副本值  
注意：后者如果没有mutable关键字，是不能修改捕获的变量，就算是副本也不行。  


## Redis 
+ redis rdb分析big key  
rdb -c memory -l 10 /data/redis_rdb/dump7000.rdb

## Misc
+ 夜神模拟器logcat调试app日志  
打开android sdk/toos/lib/monitor/monitor.exe  
运行夜神安装目录下的nox_adb.exe connect 127.0.0.1:62001  
