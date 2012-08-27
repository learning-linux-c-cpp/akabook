gdb
========

程序中除了一目了然的Bug之外都需要一定的调试手段来分析到底错在哪。到目前为止我们的调试手段只有一种：根据程序执行时的出错现象假设错误原因，然后在代码中适当的位置插入 ``printf`` ，执行程序并分析打印结果，如果结果和预期的一样，就基本上证明了自己假设的错误原因，就可以动手修Bug了，如果结果和预期的不一样，就根据结果做进一步的假设和分析。

本章我们介绍一种很强大的调试工具 :command:`gdb` ，可以完全操控程序的运行，使得程序就像你手里的玩具一样，叫它走就走，叫它停就停，并且随时可以查看程序中所有的内部状态，比如各变量的值、传给函数的参数、当前执行的代码行等。掌握了 :command:`gdb` 的用法之后，调试手段就更加丰富了。但要注意，即使调试手段丰富了，调试的基本思想仍然是“分析现象→假设错误原因→产生新的现象去验证假设”这样一个循环，根据现象如何假设错误原因，以及如何设计新的现象去验证假设，这都需要非常严密的分析和思考，如果因为手里有了强大的工具就滥用而忽略了分析过程，往往会治标不治本地修正Bug，导致一个错误现象消失了但Bug仍然存在，甚至是把程序越改越错。本章通过初学者易犯的几个错误实例来讲解如何使用 :command:`gdb` 调试程序，在每个实例后面总结一部分常用的 :command:`gdb` 命令。

单步执行和跟踪函数调用
---------------------------

看下面的程序：

.. code-block:: c
   :linenos:

   #include <stdio.h>

   int add_range(int low, int high)
   {
           int i, sum;
           for (i = low; i <= high; i++)
                   sum = sum + i;
           return sum;
   }

   int main(void)
   {
           int result[1000];
           result[0] = add_range(1, 10);
           result[1] = add_range(1, 100);
           printf("result[0]=%d\nresult[1]=%d\n", result[0], result[1]);
           return 0;
   }


``add_range`` 函数从 ``low`` 加到 ``high`` ，在 ``main`` 函数中首先从1加到10，把结果保存下来，然后从1加到100，再把结果保存下来，最后打印的两个结果是::

   result[0]=55
   result[1]=5105

第一个结果正确，第二个结果显然不正确 [#]_ ，在小学我们就听说过高斯小时候的故事，从1加到100应该是5050。一段代码，第一次运行结果是对的，第二次运行却不对，这是很常见的一类错误现象，这种情况一方面要怀疑代码，另一方面更要怀疑数据：第一次和第二次运行的都是同一段代码，如果代码是错的，那第一次的结果为什么能对呢？所以很可能是第二次运行时相关的状态和数据错了，错误的数据导致了错误的结果。在动手调试之前，读者先试试只看代码能不能看出错误原因，只要前面几章学得扎实就应该能看出来。

.. [#] 如果你编译运行这个程序的环境和我的环境（Ubuntu 12.04 LTS 32位x86）不同，也许在你的机器上跑不出这个结果，那也没关系，重要的是学会本章介绍的思想方法。另外你也可以尝试修改程序，总有办法得到类似的结果，上例中故意定义了一个很大的数组 ``result[1000]`` ，修改数组的大小就会改变各局部变量的存储空间的位置，运行结果就可能会不同。

在编译时要加上 :option:`-g` 选项，生成的可执行文件才能用 :command:`gdb` 进行源码级调试::

   $ gcc -g main.c -o main
   $ gdb main
   GNU gdb (Ubuntu/Linaro 7.4-2012.02-0ubuntu2) 7.4-2012.02
   Copyright (C) 2012 Free Software Foundation, Inc.
   License GPLv3+: GNU GPL version 3 or later <http://gnu.org/licenses/gpl.html>
   This is free software: you are free to change and redistribute it.
   There is NO WARRANTY, to the extent permitted by law.  Type "show copying"
   and "show warranty" for details.
   This GDB was configured as "i686-linux-gnu".
   For bug reporting instructions, please see:
   <http://bugs.launchpad.net/gdb-linaro/>...
   Reading symbols from /home/akaedu/main...done.
   (gdb) 

:option:`-g` 选项的作用是在可执行文件中加入源文件的信息，即可执行文件 :file:`main` 中的第几条机器指令对应源文件 :file:`main.c` 的第几行，但并不是把整个源文件嵌入到可执行文件中，所以在调试时必须保证 :command:`gdb` 能找到源文件 :file:`main.c` 。 :command:`gdb` 提供一个类似Shell的命令行环境，上面的 ``(gdb)`` 就是提示符，在这个提示符下输入 ``help`` 可以查看命令的类别::

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
   Type "help all" for the list of all commands.
   Type "help" followed by command name for full documentation.
   Type "apropos word" to search for commands related to "word".
   Command name abbreviations are allowed if unambiguous.

也可以进一步查看某一类别中有哪些命令，例如查看 ``files`` 类别下有哪些命令可用::

   (gdb) help files
   Specifying and examining files.

   List of commands:

   add-symbol-file -- Load symbols from FILE
   add-symbol-file-from-memory -- Load the symbols out of memory from a dynamically loaded object file
   cd -- Set working directory to DIR for debugger and program being debugged
   core-file -- Use FILE as core dump for examining memory and registers
   directory -- Add directory DIR to beginning of search path for source files
   edit -- Edit specified file or function
   exec-file -- Use FILE as program for getting contents of pure memory
   file -- Use FILE as program to be debugged
   forward-search -- Search for regular expression (see regex(3)) from last line listed
   generate-core-file -- Save a core file with the current state of the debugged process
   list -- List specified function or line
   ...

现在试试用 ``list`` 命令从第一行开始列出源代码::

   (gdb) list 1
   1	#include <stdio.h>
   2	
   3	int add_range(int low, int high)
   4	{
   5		int i, sum;
   6		for (i = low; i <= high; i++)
   7			sum = sum + i;
   8		return sum;
   9	}
   10  

一次只列10行，如果要从第11行开始继续列源代码可以再输入一次::

   (gdb) list

也可以什么都不输直接敲回车， :command:`gdb` 提供了一个很方便的功能，在提示符下直接敲回车表示重复上一条命令::

   (gdb) （直接回车）
   11	int main(void)
   12	{
   13		int result[1000];
   14		result[0] = add_range(1, 10);
   15		result[1] = add_range(1, 100);
   16		printf("result[0]=%d\nresult[1]=%d\n", result[0], result[1]);
   17		return 0;
   18	}

:command:`gdb` 的很多常用命令有简写形式，例如 ``list`` 命令可以写成 ``l`` ，要列一个函数的源代码也可以用函数名做参数::

   (gdb) l add_range
   1	#include <stdio.h>
   2	
   3	int add_range(int low, int high)
   4	{
   5		int i, sum;
   6		for (i = low; i <= high; i++)
   7			sum = sum + i;
   8		return sum;
   9	}
   10	

现在退出 :command:`gdb` 的环境::

   (gdb) quit

我们做一个实验，把源代码改名或移到别处再用 :command:`gdb` 调试，这样就列不出源代码了::

   $ mv main.c mian.c
   $ gdb main
   ...
   (gdb) l
   5	main.c: No such file or directory.

可见 :command:`gcc` 的 :option:`-g` 选项并不是把源代码嵌入到可执行文件中，在调试时也需要源文件。现在把源代码恢复原样，我们继续调试。首先用 ``start`` 命令开始执行程序::

   $ gdb main
   ...
   (gdb) start
   Temporary breakpoint 1 at 0x8048415: file main.c, line 14.
   Starting program: /home/akaedu/main 

   Temporary breakpoint 1, main () at main.c:14
   14		result[0] = add_range(1, 10);
   (gdb) 

:command:`gdb` 停在 ``main`` 函数中变量定义之后的第一条语句处等待我们发命令（ :command:`gdb` 在提示符之前最后列出的语句总是“即将执行的下一条语句”）。我们可以用 ``next`` 命令（简写为 ``n`` ）控制这些语句一条一条地执行::

   (gdb) n
   15		result[1] = add_range(1, 100);
   (gdb) （直接回车）
   16		printf("result[0]=%d\nresult[1]=%d\n", result[0], result[1]);
   (gdb) （直接回车）
   result[0]=55
   result[1]=5105
   17		return 0;

用 ``n`` 命令依次执行两行赋值语句和一行打印语句，在执行打印语句时结果立刻打出来了，然后停在 ``return`` 语句之前等待我们发命令。虽然我们完全控制了程序的执行，但仍然看不出哪里错了，因为错误不在 ``main`` 函数中而在 ``add_range`` 函数中，现在用 ``start`` 命令重新来过，这次用 ``step`` 命令（简写为 ``s`` ）钻进 ``add_range`` 函数中去跟踪执行::

   (gdb) start
   The program being debugged has been started already.
   Start it from the beginning? (y or n) y
   Temporary breakpoint 2 at 0x8048415: file main.c, line 14.
   Starting program: /home/akaedu/main 

   Temporary breakpoint 2, main () at main.c:14
   14		result[0] = add_range(1, 10);
   (gdb) s
   add_range (low=1, high=10) at main.c:6
   6		for (i = low; i <= high; i++)

这次停在了 ``add_range`` 函数中变量定义之后的第一条语句处。在函数中有几种查看状态的办法， ``backtrace`` 命令（简写为 ``bt`` ）可以查看函数调用的栈帧：

(gdb) bt
#0  add_range (low=1, high=10) at main.c:6
#1  0x08048429 in main () at main.c:14

可见当前的 ``add_range`` 函数是被 ``main`` 函数调用的， ``main`` 传进来的参数是 ``low=1, high=10`` 。 ``main`` 函数的栈帧编号为1， ``add_range`` 的栈帧编号为0。现在可以用 ``info`` 命令（简写为 ``i`` ）查看 ``add_range`` 函数局部变量的值::

   (gdb) i locals
   i = 0
   sum = 0

如果想查看 ``main`` 函数当前局部变量的值也可以做到，先用 ``frame`` 命令（简写为 ``f`` ）选择1号栈帧然后再查看局部变量::

   (gdb) f 1
   #1  0x08048429 in main () at main.c:14
   14		result[0] = add_range(1, 10);
   (gdb) i locals 
   result = {0 <repeats 471 times>, 1184572, 0 <repeats 11 times>, -1207961512, -1073746088, 1249268, -1073745624, 1142336, 
   ...

注意到 ``result`` 数组中很多元素的值是杂乱无章的，我们知道未经初始化的局部变量具有不确定的值，到目前为止一切正常。用 ``s`` 或 ``n`` 往下走几步，然后用 ``print`` 命令（简写为 ``p`` ）打印出变量 ``sum`` 的值::

   (gdb) s
   7			sum = sum + i;
   (gdb) （直接回车）
   6		for (i = low; i <= high; i++)
   (gdb) （直接回车）
   7			sum = sum + i;
   (gdb) （直接回车）
   6		for (i = low; i <= high; i++)
   (gdb) p sum
   $1 = 3

第一次循环 ``i`` 是1，第二次循环 ``i`` 是2，加起来是3，没错。这里的 ``$1`` 表示 :command:`gdb` 保存着这些中间结果，$后面的编号会自动增长，在命令中可以用 ``$1`` 、 ``$2`` 、 ``$3`` 等编号代替相应的值。由于我们本来就知道第一次调用的结果是正确的，再往下跟也没意义了，可以用 ``finish`` 命令让程序一直运行到从当前函数返回为止::

   (gdb) finish
   Run till exit from #0  add_range (low=1, high=10) at main.c:6
   0x08048429 in main () at main.c:14
   14		result[0] = add_range(1, 10);
   Value returned is $2 = 55

返回值是55，当前正准备执行赋值操作，用 ``n`` 命令执行赋值操作后查看 ``result`` 数组::

   (gdb) n
   15		result[1] = add_range(1, 100);
   (gdb) p result
   $3 = {55, 0 <repeats 470 times>, 1184572, 0 <repeats 11 times>, -1207961512, -1073746088, 1249268, -1073745624, 1142336, 
   ...

第一个值55确实赋给了 ``result`` 数组的第0个元素。下面用 ``s`` 命令进入第二次 ``add_range`` 调用，进入之后首先查看参数和局部变量::

   (gdb) s
   add_range (low=1, high=100) at main.c:6
   6		for (i = low; i <= high; i++)
   (gdb) bt
   #0  add_range (low=1, high=100) at main.c:6
   #1  0x08048441 in main () at main.c:15
   (gdb) i locals 
   i = 11
   sum = 55

由于局部变量 ``i`` 和 ``sum`` 没初始化，所以具有不确定的值，又由于两次调用是挨着的， ``i`` 和 ``sum`` 正好取了上次调用时的值，回顾一下我们讲过的 :ref:`验证局部变量存储空间的分配和释放 <func.verifylocals>` 那个例子，其实和现在这个例子是一样的道理，只不过我这次举的例子设法让局部变量 ``sum`` 在第一次调用时初值为0而第二次调用时初值不为0。 ``i`` 的初值不确定倒没关系，在 ``for`` 循环中首先会把 ``i`` 赋值为 ``low`` ，但 ``sum`` 如果初值不是0，累加得到的结果就错了。好了，我们已经找到错误原因，可以退出 :command:`gdb` 修改源代码了。如果我们不想浪费这次调试机会，可以在 :command:`gdb` 中马上把 ``sum`` 的初值改为0继续运行，看看这一处改了之后还有没有别的Bug::

   (gdb) set var sum=0
   (gdb) finish
   Run till exit from #0  add_range (low=1, high=100) at main.c:6
   0x08048441 in main () at main.c:15
   15		result[1] = add_range(1, 100);
   Value returned is $4 = 5050
   (gdb) n
   16		printf("result[0]=%d\nresult[1]=%d\n", result[0], result[1]);
   (gdb) （直接回车）
   result[0]=55
   result[1]=5050
   17		return 0;

这样结果就对了。修改变量的值除了用 ``set`` 命令之外也可以用 ``print`` 命令，因为 ``print`` 命令后面跟的是表达式，而我们知道赋值和函数调用也都是表达式，所以也可以用 ``print`` 命令修改变量的值或者调用函数::

   (gdb) p result[2]=33
   $5 = 33
   (gdb) p printf("result[2]=%d\n", result[2])
   result[2]=33
   $6 = 13

我们讲过， ``printf`` 的返回值表示实际打印的字符数，所以 ``$6`` 的结果是13。最后总结一下本节用到的 :command:`gdb` 命令：

.. table:: gdb基本命令1

   ======================   =================================================
   命令                      描述
   ======================   =================================================
   backtrace（或bt）         查看各级函数调用及参数
   finish                    连续运行到当前函数返回为止，然后停下来等待命令
   frame（或f） 帧编号        选择栈帧
   info（或i） locals        查看当前栈帧局部变量的值
   list（或l）               列出源代码，接着上次的位置往下列，每次列10行
   list 行号                 列出从第几行开始的源代码
   list 函数名               列出某个函数的源代码
   next（或n）               执行下一行语句
   print（或p）              打印表达式的值，通过表达式可以修改变量的值或者调用函数
   quit（或q）               退出 :command:`gdb` 调试环境
   set var                   修改变量的值
   start                     开始执行程序，停在 ``main`` 函数第一行语句前面等待命令
   step（或s）               执行下一行语句，如果有函数调用则进入到函数中
   ======================   =================================================

.. rubric:: 习题

#. 用 :command:`gdb` 一步一步跟踪 :ref:`func2.recurse` 讲的 ``factorial`` 函数，对照着 :ref:`func2.factorial` 查看各层栈帧的变化情况，练习本节所学的各种 :command:`gdb` 命令。

.. _gdb.breakpoint:

断点
----------

看以下程序：

.. code-block:: c
   :linenos:

   #include <stdio.h>

   int main(void)
   {
           int sum = 0, i = 0;
           char input[5];

           while (1) {
                   scanf("%s", input);
                   for (i = 0; input[i] != '\0'; i++)
                           sum = sum*10 + input[i] - '0';
                   printf("input=%d\n", sum);
           }
           return 0;
   }


这个程序的作用是：首先从键盘读入一串数字存到字符数组 ``input`` 中，然后转换成整型存到 ``sum`` 中，然后打印出来，一直这样循环下去。 ``scanf("%s", input);`` 这个调用的功能是等待用户输入一个字符串并回车， ``scanf`` 把其中第一段非空白（非空格、Tab、换行）的字符串保存到 ``input`` 数组中，并自动在末尾添加 ``'\0'`` 。接下来的循环从左到右扫描字符串并把每个数字累加到结果中，例如输入是 ``"2345"`` ，则循环累加的过程是(((0×10+2)×10+3)×10+4)×10+5=2345。注意字符型的 ``'2'`` 要减去 ``'0'`` 的ASCII码才能转换成整数值2。下面编译运行程序看看有什么问题::

   $ gcc main.c -g -o main
   $ ./main 
   123
   input=123
   234
   input=123234
   ^C（按Ctrl-C退出程序）
   $

又是这种现象，第一次是对的，第二次就不对。可是这个程序我们并没有忘了赋初值，不仅 ``sum`` 赋了初值，连不必赋初值的 ``i`` 都赋了初值。读者先试试只看代码能不能看出错误原因。下面来调试::

   $ gdb main
   ...
   (gdb) start
   Temporary breakpoint 1 at 0x804843d: file main.c, line 4.
   Starting program: /home/akaedu/main 

   Temporary breakpoint 1, main () at main.c:4
   4	{
   (gdb) n
   5	        int sum = 0, i = 0;

有了上一次的经验， ``sum`` 被列为重点怀疑对象，我们可以用 ``display`` 命令使得每次停下来的时候都显示当前 ``sum`` 的值，然后继续往下走::

   (gdb) display sum
   1: sum = 1466933
   (gdb) n
   9	                scanf("%s", input);
   1: sum = 0
   (gdb) 
   123
   10	                for (i = 0; input[i] != '\0'; i++)
   1: sum = 0

.. index:: d断点, Breakpoint

``undisplay`` 命令可以取消跟踪显示，变量 ``sum`` 的编号是1，可以用 ``undisplay 1`` 命令取消它的跟踪显示。这个循环应该没有问题，因为上面第一次输入时打印的结果是正确的。如果不想一步一步走这个循环，可以用 ``break`` 命令（简写为 ``b`` ）在第9行设一个断点（Breakpoint）::

   (gdb) l
   5		int sum = 0, i = 0;
   6		char input[5];
   7	
   8		while (1) {
   9			scanf("%s", input);
   10			for (i = 0; input[i] != '\0'; i++)
   11				sum = sum*10 + input[i] - '0';
   12			printf("input=%d\n", sum);
   13		}
   14		return 0;
   (gdb) b 9
   Breakpoint 2 at 0x8048459: file main.c, line 9.

``break`` 命令的参数也可以是函数名，表示在某个函数开头设断点。现在用 ``continue`` 命令（简写为 ``c`` ）连续运行而非单步运行，程序到达断点会自动停下来，这样就可以停在下一次循环的开头::

   (gdb) c
   Continuing.
   input=123

   Breakpoint 2, main () at main.c:9
   9	                scanf("%s", input);
   1: sum = 123

然后输入新的字符串准备转换::

   (gdb) n
   234
   10	                for (i = 0; input[i] != '\0'; i++)
   1: sum = 123

问题暴露出来了，新的转换应该再次从0开始累加，而 ``sum`` 现在已经是123了，原因在于新的循环没有把 ``sum`` 归零。可见断点有助于快速跳过没有问题的代码，然后在有问题的代码上慢慢走慢慢分析，“断点加单步”是使用调试器的基本方法。至于应该在哪里设置断点，怎么知道哪些代码可以跳过而哪些代码要慢慢走，也要通过对错误现象的分析和假设来确定，以前我们用 ``printf`` 打印中间结果时也要分析应该在哪里插入 ``printf`` ，打印哪些中间结果，调试的基本思路是一样的。一次调试可以设置多个断点，用 ``info`` 命令可以查看已经设置的断点::

   (gdb) b 12
   Breakpoint 3 at 0x80484b2: file main.c, line 12.
   (gdb) i breakpoints 
   Num     Type           Disp Enb Address    What
   2       breakpoint     keep y   0x08048459 in main at main.c:9
           breakpoint already hit 1 time
   3       breakpoint     keep y   0x080484b2 in main at main.c:12

每个断点都有一个编号，可以用编号指定删除某个断点::

   (gdb) delete breakpoints 2
   (gdb) i breakpoints 
   Num     Type           Disp Enb Address    What
   3       breakpoint     keep y   0x080484b2 in main at main.c:12

有时候一个断点暂时不用可以禁用掉而不必删除，这样以后想用的时候可以直接启用，而不必重新从代码里找应该在哪一行设断点::

   (gdb) disable breakpoints 3
   (gdb) i breakpoints 
   Num     Type           Disp Enb Address    What
   3       breakpoint     keep n   0x080484b2 in main at main.c:12
   (gdb) enable 3
   (gdb) i breakpoints 
   Num     Type           Disp Enb Address    What
   3       breakpoint     keep y   0x080484b2 in main at main.c:12
   (gdb) delete  breakpoints 
   Delete all breakpoints? (y or n) y
   (gdb) i breakpoints 
   No breakpoints or watchpoints.

:command:`gdb` 的断点功能非常灵活，还可以设置断点在满足某个条件时才激活，例如我们仍然在循环开头设置断点，但是仅当 ``sum`` 不等于0时才中断，然后用 ``run`` 命令（简写为 ``r`` ）重新从程序开头连续运行::

   (gdb) break 9 if sum != 0
   Breakpoint 4 at 0x8048459: file main.c, line 9.
   (gdb) i breakpoints 
   Num     Type           Disp Enb Address    What
   4       breakpoint     keep y   0x08048459 in main at main.c:9
           stop only if sum != 0
   (gdb) r
   The program being debugged has been started already.
   Start it from the beginning? (y or n) y
   Starting program: /home/akaedu/main 
   123
   input=123

   Breakpoint 4, main () at main.c:9
   9	                scanf("%s", input);
   1: sum = 123

结果是第一次执行 ``scanf`` 之前没有中断，第二次却中断了。总结一下本节用到的 :command:`gdb` 命令：

.. table:: gdb基本命令2

   ============================   ========================================
   命令	                          描述
   ============================   ========================================
   break（或b） 行号                在某一行设置断点
   break 函数名                    在某个函数开头设置断点
   break ... if ...               设置条件断点
   continue（或c）                 从当前位置开始连续运行程序
   delete breakpoints 断点号       删除断点
   display 变量名                  跟踪查看某个变量，每次停下来都显示它的值
   disable breakpoints 断点号      禁用断点
   enable 断点号                   启用断点
   info（或i） breakpoints         查看当前设置了哪些断点
   run（或r）                      从头开始连续运行程序
   undisplay 跟踪显示号             取消跟踪显示
   ============================   ========================================

.. rubric:: 习题

#. 看下面的程序：

   .. code-block:: c
      :linenos:

      #include <stdio.h>

      int main(void)
      {
              int i;
              char str[6] = "hello";
              char reverse_str[6] = "";

              printf("%s\n", str);
              for (i = 0; i < 5; i++)
                      reverse_str[5-i] = str[i];
              printf("%s\n", reverse_str);
              return 0;
      }

   首先用字符串 ``"hello"`` 初始化一个字符数组 ``str`` （算上 ``'\0'`` 共6个字符）。然后用空字符串 ``""`` 初始化一个同样长的字符数组 ``reverse_str`` ，相当于所有元素用 ``'\0'`` 初始化。然后打印 ``str`` ，把 ``str`` 倒序存入 ``reverse_str`` ，再打印 ``reverse_str`` 。然而结果并不正确::

      $ ./main 
      hello

   我们本来希望 ``reverse_str`` 打印出来是 ``olleh`` ，结果打出来一个空行。重点怀疑对象肯定是循环，那么简单验算一下， ``i=0`` 时， ``reverse_str[5]=str[0]`` ，也就是 ``'h'`` ， ``i=1`` 时， ``reverse_str[4]=str[1]`` ，也就是 ``'e'`` ，依此类推，i=0,1,2,3,4，共5次循环，正好把h,e,l,l,o五个字母给倒过来了，哪里不对了？请用 :command:`gdb` 跟踪循环，找出错误原因并改正。

观察点
-----------

继续修改上一节的程序。经过调试我们得出结论，对于这个程序来说， ``sum`` 赋不赋初值不重要，重要的是在 ``while (1)`` 循环体的开头加上 ``sum = 0;`` ，这才能保证每次循环从0开始累加。我们把程序改成这样：

.. code-block:: c
   :linenos:

   #include <stdio.h>

   int sum = 0, i;
   char input[5];

   int main(void)
   {
           while (1) {
                   sum = 0;
                   scanf("%s", input);
                   for (i = 0; input[i] != '\0'; i++)
                           sum = sum*10 + input[i] - '0';
                   printf("input=%d\n", sum);
           }
           return 0;
   }

在这里我故意把 ``sum`` 、 ``i`` 、 ``input`` 定义成全局变量， ``sum`` 赋初值而 ``i`` 和 ``input`` 不赋初值，这是为了比较容易产生本节要讲的错误现象。还是那句话，如果你的运行环境和我不同，在你机器上可能跑不出书上说的结果。你可以先看书，在理解了基本原理之后自己改改程序看能不能跑出类似的结果：变量定义在全局还是局部作用域，在定义时是否初赋了初值，这些都会影响变量所占的存储空间的位置，从而影响本程序的运行结果。

使用 ``scanf`` 函数是非常凶险的，即使修正了上一节的Bug也还存在很多问题。如果输入的字符串超长了会怎么样？我们知道数组访问越界是不会被检查的，所以 ``scanf`` 会把 ``input`` 数组写越界。现象是这样的::

   $ ./main 
   1234
   input=1234
   1234567
   input=1234567
   12345678
   input=123456740

输入1234567其实已经访问越界了，但程序还能给出正确结果。而输入12345678时程序给出一个非常诡异的结果，下面我们用调试器看看这个诡异的结果是怎么出来的::

   $ gdb main
   ...
   (gdb) start
   Temporary breakpoint 1 at 0x804843d: file main.c, line 9.
   Starting program: /home/akaedu/main 

   Temporary breakpoint 1, main () at main.c:9
   9	                sum = 0;
   (gdb) n
   10	                scanf("%s", input);
   (gdb) （直接回车）
   12345678
   11	                for (i = 0; input[i] != '\0'; i++)
   (gdb) p input
   $1 = "12345"

在这里 :command:`gdb` 知道 ``input`` 数组的长度是5，所以用 ``p`` 命令查看时只显示5个字符。我们换一种办法查看就可以看到其实已经写越界了::

   (gdb) p printf("%x %x %x %x %x %x %x %x %x\n", input[0], input[1], input[2], input[3], input[4], input[5], input[6], input[7], input[8])
   31 32 33 34 35 36 37 38 0
   $2 = 26

这条命令从 ``input`` 数组的第一个字节开始连续打印9个字节，打印的正是 ``'1'`` 到 ``'8'`` 的十六进制ASCII码，还有一个 ``'\0'`` ，所以 ``scanf`` 实际上写越界了四个字符：``'6'`` 、 ``'7'`` 、 ``'8'`` 、 ``'\0'`` 。 ``printf`` 的转换说明 ``%x`` 表示按16进制打印。

根据运行结果“123456740”，用户输入的前7个字符转成数字都没错，第8个错了，也就是 ``i`` 从0到6的循环都没错，我们设一个条件断点从 ``i`` 等于7开始单步调试::

   (gdb) l
   6	int main(void)
   7	{
   8	        while (1) {
   9	                sum = 0;
   10	                scanf("%s", input);
   11	                for (i = 0; input[i] != '\0'; i++)
   12	                        sum = sum*10 + input[i] - '0';
   13	                printf("input=%d\n", sum);
   14	        }
   15	        return 0;
   (gdb) b 12 if i == 7
   Breakpoint 2 at 0x8048468: file main.c, line 12.
   (gdb) c
   Continuing.

   Breakpoint 2, main () at main.c:12
   12	                        sum = sum*10 + input[i] - '0';
   (gdb) p sum
   $3 = 1234567

现在 ``sum`` 是1234567没错，我们推测即将进行的下一步计算肯定要出错，调试的结果出乎意料，下一步计算并没有出错::

   (gdb) p input[i]
   $4 = 56 '8'
   (gdb) n
   11	                for (i = 0; input[i] != '\0'; i++)
   (gdb) p sum
   $5 = 12345678

``input[i]`` 是 ``'8'`` ，减去 ``'0'`` 等于8，把 ``sum`` 的当前值1234567乘以10再加上8，确实得到了12345678。那为什么打印的结果却不是这一步算出的12345678呢？只有一个解释：这一步计算之后并没有跳出循环去执行 ``printf`` ，而是继续下一轮循环::

   (gdb) n
   12	                        sum = sum*10 + input[i] - '0';
   (gdb) p i
   $6 = 8
   (gdb) p input[i]
   $7 = 8 '\b'
   (gdb) n
   11	                for (i = 0; input[i] != '\0'; i++)
   (gdb) p sum
   $8 = 123456740
   (gdb) n
   13	                printf("input=%d\n", sum);
   (gdb) p i
   $9 = 9
   (gdb) p input[9]
   $10 = 0 '\000'

先前我们明明打印出 ``input[8]`` 是 ``'\0'`` ，什么时候变成 ``'\b'`` 的呢？这一变，循环的控制条件 ``input[8] != '\0'`` 又得到满足了，原本应该跳出循环的，现在又进循环了，把sum累加成了12345678*10 + '\b' - '0' = 123456740 （ ``'\b'`` 的ASCII码是8， ``'0'`` 的ASCII码是48）。然后 ``input[9]`` 确实是0，跳出循环，打印，终于得出了那个诡异的结果！

现在我们要弄清楚 ``input[8]`` 到底是什么时候变的，可以用观察点（Watchpoint）来跟踪。我们知道断点是当程序执行到某一代码行时中断，而观察点是当程序访问某个存储单元时中断。如果我们不知道某个存储单元是被哪一行代码改动的，观察点就非常有用了。下面删除原来设的断点，从头执行程序，重复上次的输入，用 ``watch`` 命令设置观察点，跟踪 ``input[8]`` 的存储单元::

   (gdb) delete breakpoints 
   Delete all breakpoints? (y or n) y
   (gdb) start
   The program being debugged has been started already.
   Start it from the beginning? (y or n) y
   Temporary breakpoint 3 at 0x804843d: file main.c, line 9.
   Starting program: /home/akaedu/main 

   Temporary breakpoint 3, main () at main.c:9
   9	                sum = 0;
   (gdb) n
   10	                scanf("%s", input);
   (gdb) （直接回车）
   12345678
   11	                for (i = 0; input[i] != '\0'; i++)
   (gdb) watch input[8]
   Hardware watchpoint 4: input[8]
   (gdb) i watchpoints 
   Num     Type           Disp Enb Address    What
   4       hw watchpoint  keep y              input[8]
   (gdb) c
   Continuing.
   Hardware watchpoint 4: input[8]

   Old value = 0 '\000'
   New value = 1 '\001'
   0x0804849f in main () at main.c:11
   11	                for (i = 0; input[i] != '\0'; i++)
   (gdb) c
   Continuing.
   Hardware watchpoint 4: input[8]

   Old value = 1 '\001'
   New value = 2 '\002'
   0x0804849f in main () at main.c:11
   11	                for (i = 0; input[i] != '\0'; i++)

已经很明显了，每次都是回到 ``for`` 循环开头的时候改变了 ``input[8]`` 的值，而且是每次加1－－这不就是循环变量 ``i`` 么？原来循环变量 ``i`` 就位于 ``input[8]`` 的位置。 ``input[5]`` 、 ``input[6]`` 、 ``input[7]`` 虽然也是访问越界，但还不算严重，反正也没有别的变量占用这块存储空间，而 ``input[8]`` 这个访问越界就严重了，直接访问到变量 ``i`` 的头上了。其实用 ``x`` 命令可以清楚地看到这一点，只不过为了防止“剧透”我一开始没有这么做::

   (gdb) x/12bx input
   0x804a024 <input>:	0x31	0x32	0x33	0x34	0x35	0x36	0x37	0x38
   0x804a02c <i>:	0x02	0x00	0x00	0x00

``x`` 命令打印指定的存储单元里保存的内容，后缀 ``8bx`` 是打印格式，12表示打印12组，b表示每个字节一组，x表示按十六进制格式打印 [#]_ ，我们可以看到在 ``input`` 的存储单元的起始位置加8个字节处正是变量 ``i`` 的存储单元。

.. [#] 打印结果最左边的一长串数字是内存地址，在 :ref:`arch.memaddr` 详细解释，目前可以无视。

修正这个Bug对初学者来说有一定难度。如果你发现了这个Bug却没想到数组访问越界这一点，也许一时想不出原因，就会先去处理另外一个更容易修正的Bug：如果输入的不是数字而是字母或别的符号也能算出结果来，这显然是不对的，可以在循环中加上判断条件检查非法字符。

.. code-block:: c
   :linenos:

   while (1) {
           sum = 0;
           scanf("%s", input);
           for (i = 0; input[i] != '\0'; i++) {
                   if (input[i] < '0' || input[i] > '9') {
                           printf("Invalid input!\n");
                           sum = -1;
                           break;
                   }
                   sum = sum*10 + input[i] - '0';
           }
           printf("input=%d\n", sum);
   }

然后你会惊喜地发现，不仅输入字母会报错，输入超长也会报错::

   $ ./main
   123a
   Invalid input!
   input=-1
   dead
   Invalid input!
   input=-1
   1234578
   Invalid input!
   input=-1
   1234567890abcdef
   Invalid input!
   input=-1
   23
   input=23

似乎是两个Bug一起解决掉了，但这是治标不治本的解决方法。看起来输入超长的错误是不出现了，但只要没有找到根本原因就不可能真的解决掉，等到条件一变，它可能又冒出来了，在下一节你会看到它又以一种新的形式冒出来了。现在请思考一下为什么加上检查非法字符的代码之后输入超长也会报错。

最后总结一下本节用到的 :command:`gdb` 命令：

.. table:: gdb基本命令3

   ==========================   =============================================================================
   命令                          描述
   ==========================   =============================================================================
   watch                        设置观察点
   info（或i） watchpoints       查看当前设置了哪些观察点
   x                            从某个位置开始打印存储单元的内容，全部当成字节来看，而不区分哪个字节属于哪个变量
   ==========================   =============================================================================

.. _gdb.segfault:

程序崩溃
----------------

如果程序运行时出现段错误，用 :command:`gdb` 可以很容易定位到究竟是哪一行引发的段错误，例如这个小程序：

.. code-block:: c
   :linenos:

   #include <stdio.h>

   int main(void)
   {
           int man = 0;
           scanf("%d", man);
           return 0;
   }


调试过程如下::

   $ gdb main
   ...

   (gdb) r
   Starting program: /home/akaedu/main 
   123

   Program received signal SIGSEGV, Segmentation fault.
   0x00180a93 in _IO_vfscanf () from /lib/i386-linux-gnu/libc.so.6
   (gdb) bt
   #0  0x00180a93 in _IO_vfscanf () from /lib/i386-linux-gnu/libc.so.6
   #1  0x0018747b in __isoc99_scanf () from /lib/i386-linux-gnu/libc.so.6
   #2  0x0804842a in main () at main.c:6

在 :command:`gdb` 中运行，遇到段错误会自动停下来，这时可以用命令查看当前执行到哪一行代码了。 :command:`gdb` 显示段错误出现在 ``_IO_vfscanf`` 函数中，用 ``bt`` 命令可以看到这个函数是被 ``main.c`` 的第6行间接调用的，也就是 ``scanf`` 这行代码引发的段错误。仔细观察程序发现是 ``man`` 前面少了个&。

继续调试上一节的程序，上一节最后提出修正Bug的方法是在循环中加上判断条件，如果不是数字就报错退出，结果是不仅输入非法字符可以报错退出，输入超长的字符串也会报错退出。表面上看这个程序无论怎么运行都不出错了，但假如我们把 ``while (1)`` 循环去掉，每次执行程序只转换一个数：

.. code-block:: c
   :linenos:

   #include <stdio.h>

   int main(void)
   {
           int sum = 0, i = 0;
           char input[5];

           scanf("%s", input);
           for (i = 0; input[i] != '\0'; i++) {
                   if (input[i] < '0' || input[i] > '9') {
                           printf("Invalid input!\n");
                           sum = -1;
                           break;
                   }
                   sum = sum*10 + input[i] - '0';
           }
           printf("input=%d\n", sum);

           return 0;
   }


然后输入一个超长的字符串，看看会发生什么::

   $ ./main
   12345678
   input=12345678
   *** stack smashing detected ***: ./main terminated
   ======= Backtrace: =========
   /lib/i386-linux-gnu/libc.so.6(__fortify_fail+0x45)[0xf4cdd5]
   /lib/i386-linux-gnu/libc.so.6(+0xffd8a)[0xf4cd8a]
   ./main[0x8048592]
   /lib/i386-linux-gnu/libc.so.6(__libc_start_main+0xf3)[0xe664d3]
   ./main[0x8048421]
   ======= Memory map: ========
   00138000-00158000 r-xp 00000000 08:01 394133     /lib/i386-linux-gnu/ld-2.15.so
   00158000-00159000 r--p 0001f000 08:01 394133     /lib/i386-linux-gnu/ld-2.15.so
   00159000-0015a000 rw-p 00020000 08:01 394133     /lib/i386-linux-gnu/ld-2.15.so
   00c97000-00c98000 r-xp 00000000 00:00 0          [vdso]
   00e0f000-00e2b000 r-xp 00000000 08:01 394174     /lib/i386-linux-gnu/libgcc_s.so.1
   00e2b000-00e2c000 r--p 0001b000 08:01 394174     /lib/i386-linux-gnu/libgcc_s.so.1
   00e2c000-00e2d000 rw-p 0001c000 08:01 394174     /lib/i386-linux-gnu/libgcc_s.so.1
   00e4d000-00fec000 r-xp 00000000 08:01 394153     /lib/i386-linux-gnu/libc-2.15.so
   00fec000-00fee000 r--p 0019f000 08:01 394153     /lib/i386-linux-gnu/libc-2.15.so
   00fee000-00fef000 rw-p 001a1000 08:01 394153     /lib/i386-linux-gnu/libc-2.15.so
   00fef000-00ff2000 rw-p 00000000 00:00 0 
   08048000-08049000 r-xp 00000000 08:01 439349     /home/akaedu/main
   08049000-0804a000 r--p 00000000 08:01 439349     /home/akaedu/main
   0804a000-0804b000 rw-p 00001000 08:01 439349     /home/akaedu/main
   09c65000-09c86000 rw-p 00000000 00:00 0          [heap]
   b7780000-b7781000 rw-p 00000000 00:00 0 
   b778e000-b7793000 rw-p 00000000 00:00 0 
   bfb0c000-bfb2d000 rw-p 00000000 00:00 0          [stack]
   Aborted (core dumped)

我们输入12345678，计算结果12345678都打印完了，却在最后爆出整整一屏错误信息。准确地说这是另外一种形式的程序崩溃而不是段错误，不过我们可以按同样的方法用 :command:`gdb` 调试看看::

   $ gdb main
   ...
   (gdb) r
   Starting program: /home/akaedu/main 
   12345678
   input=12345678
   *** stack smashing detected ***: /home/akaedu/main terminated
   ======= Backtrace: =========
   /lib/i386-linux-gnu/libc.so.6(__fortify_fail+0x45)[0x232dd5]
   /lib/i386-linux-gnu/libc.so.6(+0xffd8a)[0x232d8a]
   /home/akaedu/main[0x8048592]
   /lib/i386-linux-gnu/libc.so.6(__libc_start_main+0xf3)[0x14c4d3]
   /home/akaedu/main[0x8048421]
   ======= Memory map: ========
   00110000-00130000 r-xp 00000000 08:01 394133     /lib/i386-linux-gnu/ld-2.15.so
   00130000-00131000 r--p 0001f000 08:01 394133     /lib/i386-linux-gnu/ld-2.15.so
   00131000-00132000 rw-p 00020000 08:01 394133     /lib/i386-linux-gnu/ld-2.15.so
   00132000-00133000 r-xp 00000000 00:00 0          [vdso]
   00133000-002d2000 r-xp 00000000 08:01 394153     /lib/i386-linux-gnu/libc-2.15.so
   002d2000-002d4000 r--p 0019f000 08:01 394153     /lib/i386-linux-gnu/libc-2.15.so
   002d4000-002d5000 rw-p 001a1000 08:01 394153     /lib/i386-linux-gnu/libc-2.15.so
   002d5000-002d8000 rw-p 00000000 00:00 0 
   002d8000-002f4000 r-xp 00000000 08:01 394174     /lib/i386-linux-gnu/libgcc_s.so.1
   002f4000-002f5000 r--p 0001b000 08:01 394174     /lib/i386-linux-gnu/libgcc_s.so.1
   002f5000-002f6000 rw-p 0001c000 08:01 394174     /lib/i386-linux-gnu/libgcc_s.so.1
   08048000-08049000 r-xp 00000000 08:01 439349     /home/akaedu/main
   08049000-0804a000 r--p 00000000 08:01 439349     /home/akaedu/main
   0804a000-0804b000 rw-p 00001000 08:01 439349     /home/akaedu/main
   0804b000-0806c000 rw-p 00000000 00:00 0          [heap]
   b7fed000-b7fee000 rw-p 00000000 00:00 0 
   b7ffb000-b8000000 rw-p 00000000 00:00 0 
   bffdf000-c0000000 rw-p 00000000 00:00 0          [stack]

   Program received signal SIGABRT, Aborted.
   0x00132416 in __kernel_vsyscall ()
   (gdb) bt
   #0  0x00132416 in __kernel_vsyscall ()
   #1  0x001611ef in raise () from /lib/i386-linux-gnu/libc.so.6
   #2  0x00164835 in abort () from /lib/i386-linux-gnu/libc.so.6
   #3  0x0019c2fa in ?? () from /lib/i386-linux-gnu/libc.so.6
   #4  0x00232dd5 in __fortify_fail () from /lib/i386-linux-gnu/libc.so.6
   #5  0x00232d8a in __stack_chk_fail () from /lib/i386-linux-gnu/libc.so.6
   #6  0x08048592 in main () at main.c:20

:command:`gdb` 指出，错误发生在第20行。可是这一行什么都没有啊，只有表示 ``main`` 函数结束的}括号。这可以算是一条规律， **如果某个函数的局部变量发生访问越界，有可能并不立即产生段错误，而是在函数返回时产生段错误** 。

想要写出Bug-free的程序是非常不容易的，即使 ``scanf`` 读入字符串这么一个简单的函数调用都会隐藏着各种各样的错误。有些错误现象是我们暂时没法解释的，在后续章节中都会解释清楚。其实现在讲 ``scanf`` 这个函数为时过早，读者还不具备充足的基础知识，而且这个函数的用法也确实是相当复杂，要用得准确无误是挺难的，本书将在 :ref:`stdlib.formattedio` 详细解释这个函数。现在早早地引入这个函数是为了让读者可以早早地开始写有用的程序，毕竟，一个只能输出（ ``printf`` ）而不能输入（ ``scanf`` ）的程序算不上什么有用的程序。



