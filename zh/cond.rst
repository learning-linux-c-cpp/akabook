分支语句
==================

.. _cond.if:

if语句
-----------

目前我们写的简单函数中可以有多条语句，但这些语句总是从前到后顺序执行的。除了顺序执行之外，有时候我们需要检查一个条件，然后根据检查的结果执行不同的后续代码，在C语言中可以用分支语句实现，比如：

.. code-block:: c
   :linenos:

   if (x != 0) {
           printf("x is nonzero.\n");
   }

.. index:: k控制表达式, Controlling Expressoin, k控制流程, f分支, Branch

其中 ``x != 0`` 表示“x不等于0”的条件，这个表达式称为控制表达式（Controlling Expression），如果条件成立，则{}中的语句被执行，否则{}中的语句不执行，直接跳到}后面。 ``if`` 和控制表达式改变了程序的控制流程（Control Flow），不再是从前到后顺序执行，而是根据不同的条件执行不同的语句，这种控制流程称为分支（Branch）。上例中的 ``!=`` 运算符表示“不等于”，像这样的运算符有：

.. table:: 关系运算符和相等性运算符

   =========    ===========
   运算符        含义
   =========    ===========
   ==           等于
   !=           不等于
   >            大于
   <            小于
   >=           大于或等于
   <=           小于或等于
   =========    ===========

注意以下几点：

.. index:: z真, True, j假, False, x相等性运算符, Equality Operator, g关系运算符, Relational Operator

#. 这里的 ``==`` 表示数学中的相等关系，相当于数学中的=号，初学者常犯的错误是在控制表达式中把 ``==`` 写成 ``=`` ，在C语言中=号是赋值运算符，两者的含义完全不同。
#. 如果表达式所表示的比较关系成立则值为真（True），否则为假（False），在C语言中分别用 ``int`` 型的1和0表示。如果变量 ``x`` 的值是-1，那么 ``x > 0`` 这个表达式的值为0， ``x > -2`` 这个表达式的值为1。
#. 在数学中a<b<c表示b既大于a又小于c，但作为C语言表达式却不是这样。以上几种运算符都是左结合的，请读者想一下这个表达式在C语言中的求值步骤是怎样的。
#. 这些运算符的两个操作数应该是相同类型的，两边都是整型或者都是浮点型可以做比较，但两个字符串不能做比较，在 :ref:`stdlib.comparestring` 我们会介绍比较字符串的方法。
#. ``==`` 和 ``!=`` 称为相等性运算符（Equality Operator），其余四个称为关系运算符（Relational Operator），相等性运算符的优先级低于关系运算符。

总结一下， ``if (x != 0) { ... }`` 这个语句的计算顺序是：首先求 ``x != 0`` 这个表达式的值，如果值为0，就跳过{}中的语句直接执行后面的语句，如果值为1，就先执行{}中的语句，然后再执行后面的语句。事实上控制表达式取任何非0值都表示真值，因此 ``if (x) { ... }`` 和 ``if (x != 0) { ... }`` 是等价的：假设 ``x`` 的值是2，则 ``x != 0`` 的值是1，但对于 ``if`` 来说不管控制表达式的值是2还是1都表示真值。

和if语句相关的语法规则如下::

   语句 → if (控制表达式) 语句
   语句 → { 语句列表 }
   语句 → ;

.. index:: y语句块, Statement Block, k空语句, Null Statement

在C语言中，任何允许出现语句的地方既可以是由;号结尾的一条语句，也可以是由{}括起来的若干条语句或声明组成的语句块（Statement Block），语句块和上一章介绍的函数体的语法相同。注意语句块的}后面不需要加;号。如果}后面加了;号，则这个;号本身又是一条新的语句了，在C语言中一个单独的;号表示一条空语句（Null Statement）。上例的语句块中只有一条语句，其实没必要写成语句块，可以简单地写成：

.. code-block:: c
   :linenos:

   if (x != 0)
           printf("x is nonzero.\n");

``if`` 语句不一定要包含语句块，而语句块也不一定非得用在 ``if`` 语句中。语句块可以用在任何允许出现语句的地方，单独使用语句块通常是为了定义一些比函数的局部变量更“局部”的变量。例如：

.. code-block:: c
   :linenos:

   void foo(void)
   {
           int i = 0;
           {
                   int i = 1;
                   int j = 2;
                   printf("i=%d, j=%d\n", i, j);
           }
           printf("i=%d\n", i); /* cannot access j here */
   }

和函数的局部变量同样道理，每次进入语句块时为变量 ``j`` 分配存储空间，每次退出语句块时释放变量 ``j`` 的存储空间。语句块也构成一个作用域，类似 :ref:`func.scope` 分析一下可以得到如下结论：

#. 如果整个源文件是一张大纸， ``foo`` 函数是盖在上面的一张小纸，则函数中的语句块是盖在小纸上面的一张更小的纸。
#. 语句块中的变量 ``i`` 和函数的局部变量 ``i`` 是两个不同的变量，因此两次打印的 ``i`` 值是不同的。
#. 语句块中的变量 ``j`` 在退出语句块之后就没有了，因此最后一行的 ``printf`` 不能打印变量 ``j`` ，否则编译器会报错。

.. rubric:: 习题

#. 以下程序段编译能通过，执行也不出错，但是执行结果不正确（根据 :ref:`intro.debug` 的定义，这是一个语义错误），请分析一下哪里错了。还有，既然错了为什么编译能通过呢？

   .. code-block:: c
      :linenos:

      int x = -1;
      if (x > 0);
              printf("x is positive\n");

.. _cond.ifelse:

if/else语句
-------------------

.. index:: z子句, Clause

``if`` 语句还可以带一个 ``else`` 子句（Clause），例如：

.. code-block:: c
   :linenos:

   if (x % 2 == 0)
           printf("x is even\n");
   else
           printf("x is odd\n");

.. index:: q取模, Modulo, y余数, Remainder

这里的%是取模（Modulo）运算符， ``x%2`` 表示 ``x`` 除以2所得的余数（Remainder），C语言规定%运算符的两个操作数必须是整型的。两个正数相除取余数很好理解，如果操作数中有负数，结果应该是正是负呢？C99规定：如果 ``a`` 和 ``b`` 是整型， ``b`` 不等于0，则表达式 ``(a/b)*b+a%b`` 的值总是等于 ``a`` 。再结合 :ref:`expr.expression` 讲过的整数除法运算要Truncate towards Zero，可以得到一个结论： **%运算符的结果总是与被除数同号** （读者可以自己推导一下）。其他编程语言对取模运算的规定各不相同，也有规定结果和除数同号的，也有不做明确规定的。

取模运算在程序非常有用，例如上面的例子判断 ``x`` 的奇偶性，看 ``x`` 除以2的余数是不是0，如果是0则打印 ``x is even`` ，如果不是0则打印 ``x is odd`` 。读者应该能看出 ``else`` 在这里的作用了：假如从上面的例子中去掉 ``else`` ，则不管 ``x`` 是奇是偶， ``printf("x is odd\n");`` 这条语句总是被执行。

为了让这个判断和打印操作更有用，可以把它封装成一个函数：

.. code-block:: c
   :linenos:

   void print_even_or_odd(int x)
   {
           if (x % 2 == 0)
                   printf("x is even\n");
           else
                   printf("x is odd\n");
   }

**把语句封装成函数的基本步骤是：把语句放到函数体中，把变量改成函数的参数。** 这样，以后要检查一个数的奇偶性只需调一下函数即可，不必重复写这个 ``if/else`` 语句了，例如：

.. code-block:: c
   :linenos:

   print_even_or_odd(17);
   print_even_or_odd(18);

``if/else`` 语句的语法规则如下::

   语句 → if (控制表达式) 语句 else 语句

右边的“语句”既可以是一条语句，也可以是由{}括起来的语句块。一条 ``if`` 语句中包含一条子语句，一条 ``if/else`` 语句中包含两条子语句，子语句可以是任何语句或语句块，当然也可以是另外一条 ``if`` 或 ``if/else`` 语句。根据组合规则， ``if`` 或 ``if/else`` 可以嵌套使用。例如可以这样：

.. code-block:: c
   :linenos:

   if (x > 0) {
           printf("x is positive\n");
   } else {
           if (x < 0)
                   printf("x is negative\n");
           else
                   printf("x is zero\n");
   }

等价于下面这种不用语句块的写法：

.. code-block:: c
   :linenos:

   if (x > 0)
           printf("x is positive\n");
   else if (x < 0)
           printf("x is negative\n");
   else
           printf("x is zero\n");

现在问题来了：如果不用语句块，那么类似 ``if (A) if (B) C; else D;`` 形式的语句该怎么理解呢？可以理解成

.. code-block:: c
   :linenos:

   if (A)
           if (B)
                   C;
   else
           D;

也可以理解成

.. code-block:: c
   :linenos:

   if (A)
           if (B)
                   C;
           else
                   D;

.. index:: Dangling-else

在 :ref:`expr.helloworld` 讲过，C代码的缩进只是为了程序员看起来方便，实际上对编译器不起任何作用，你的代码不管写成上面哪一种缩进格式，在编译器看起来都是一样的。那么编译器到底按哪种方式理解呢？换句话说，编译器认为 ``else`` 到底是和 ``if (A)`` 配对还是和 ``if (B)`` 配对？很多编程语言的语法都有这个问题，称为Dangling-else问题。C语言规定， **else总是和它前面最近的一个if配对** ，因此应该理解成 ``else`` 和 ``if (B)`` 配对，也就是按第二种方式理解。如果你写成上面第一种缩进的格式就很危险了：你看到的是这样，而编译器理解的却是那样。如果你希望编译器按第一种方式理解，应该明确加上{}：

.. code-block:: c
   :linenos:

   if (A) {
           if (B)
                   C;
   } else
           D;

.. rubric:: 习题

#. 写两个表达式，分别取整型变量 ``x`` 的个位和十位。
#. 写一个函数，参数是整型变量 ``x`` ，功能是打印 ``x`` 的个位和十位。

.. _cond.bool:

布尔代数
------------------

在 :ref:`cond.if` 讲过，在数学上a<b<c表示b既大于a又小于c，但在C语言中不能这么写，那如果想表示这个含义该怎么写呢？可以这样：

.. code-block:: c
   :linenos:

   if (a < b) {
           if (b < c) {
                   printf("b is between a and c\n");
           }
   }

.. index:: l逻辑与, Logical AND, &, Ampersand

我们也可以用逻辑与（Logical AND）运算符来表示“两个条件同时成立”。逻辑与运算符在C语言中用两个&（Ampersand）表示，上面的语句可以改写为：

.. code-block:: c
   :linenos:

   if (a < b && b < c) {
           printf("b is between a and c\n");
   }

对于 ``a < b && b < c`` 这个控制表达式，只有“ ``a < b`` 的值非0”和“ ``b < c`` 的值非0”这两个条件同时成立，整个表达式的值才为1，否则整个表达式的值为0。换句话说，只有两个条件都为真，它们做逻辑与运算的结果才为真，只要有一个条件为假，逻辑与运算的结果就为假，如下表所示：

.. table:: AND的真值表

   = = =======
   A B A AND B
   = = =======
   0 0 0
   0 1 0
   1 0 0
   1 1 1
   = = =======

.. index:: z真值表, Truth Table, l逻辑或, Logical OR, |线, Pipe Sign, l逻辑非, Logical NOT, !号, Exclamation Mark

这种表称为真值表（Truth Table）。注意逻辑与运算的操作数以非0表示真以0表示假，而运算结果（类型是 ``int``）以1表示真以0表示假，我们忽略这些细微的差别，在表中全部以1表示真以0表示假。C语言还提供了逻辑或（Logical OR）运算符，用两个|线（Pipe Sign）表示，还有逻辑非（Logical NOT）运算符，用一个!号（Exclamation Mark）表示，它们的真值表如下：

.. table:: OR的真值表

   = = ======
   A B A OR B
   = = ======
   0 0 0
   0 1 1
   1 0 1
   1 1 1
   = = ======

.. table:: NOT的真值表

   = =====
   A NOT A
   = =====
   0 1
   1 0
   = =====

.. index:: d单目运算符, Unary Operator, s双目运算符, Binary Operator

逻辑或表示两个条件只要有一个为真，它们做逻辑或运算的结果就为真，只有两个条件都为假，逻辑或运算的结果才为假。逻辑非的作用是对原来的逻辑值取反，原来是真的就是假，原来是假的就是真。例如 ``!(x > 1)`` ，如果表达式 ``x > 1`` 的值非零，则 ``!(x > 1)`` 的值为0。逻辑非运算符只有一个操作数，称为单目运算符（Unary Operator），以前讲过的加减乘除、赋值、相等性、关系、逻辑与、逻辑或运算符都有两个操作数，称为双目运算符（Binary Operator）。

.. index:: b布尔代数, Boolean Algebra

关于逻辑运算的数学体系称为布尔代数（Boolean Algebra），以它的创始人布尔命名。在编程语言中表示真和假的数据类型叫做布尔类型，在C语言中通常用 ``int`` 型来表示，非0表示真，0表示假 [#]_ 。布尔逻辑是写程序的基本功之一，程序中的很多错误都可以归因于逻辑错误。以下是一些布尔代数的基本定理，为了简洁易读，真和假用1和0表示，AND用*号表示，OR用+号表示（从真值表可以看出AND和OR运算确实有点像乘法和加法运算），NOT用¬表示，变量x、y、z的值可能是0也可能是1，无论x、y、z怎么取值，以下等式都成立。

.. [#] C99也定义了专门的布尔类型 ``_Bool`` ，但目前没有被广泛使用。

::

   ¬¬x=x

   x*0=0
   x+1=1

   x*1=x
   x+0=x

   x*x=x
   x+x=x

   x*¬x=0
   x+¬x=1

   x*y=y*x
   x+y=y+x

   x*(y*z)=(x*y)*z
   x+(y+z)=(x+y)+z

   x*(y+z)=x*y+x*z
   x+y*z=(x+y)*(x+z)

   x+x*y=x
   x*(x+y)=x

   x*y+x*¬y=x
   (x+y)*(x+¬y)=x

   ¬(x*y)=¬x+¬y
   ¬(x+y)=¬x*¬y

   x+¬x*y=x+y
   x*(¬x+y)=x*y

   x*y+¬x*z+y*z=x*y+¬x*z
   (x+y)*(¬x+z)*(y+z)=(x+y)*(¬x+z)

除了第一个公式之外，这些公式都是每两个一组的，每组的两个公式就像对联一样：把其中一个公式中的*换成+、+换成*、0换成1、1换成0，就变成了与它对称的另一个公式。这些定理都可以用真值表来证明，更多细节可参考有关数字逻辑的教材，例如 [数字逻辑基础]_ 。我们将在本节的练习题中强化训练对这些定理的理解。

目前为止介绍的这些运算符按优先级从高到低的顺序依次是：

*  !
*  \* / %
*  \+ \-
*  > < >= <=
*  == !=
*  &&
*  ||
*  =

写一个表达式往往会用到好几种运算符，如果记不清楚运算符的优先级一定要多套括号。我们将在 :ref:`op.op` 总结C语言所有运算符的优先级和结合性。

最后要注意一点，浮点型的精度有限，不适合用 ``==`` 运算符做精确比较。以下代码可以说明问题：

.. code-block:: c
   :linenos:

   double a = 0.3 - 0.2;
   double b = 0.2 - 0.1;
   if (a == b)
           printf("Equal\n");
   else
           printf("Unequal\n");

程序的执行结果非常意外，居然是Unequal。虽然理论上 ``a`` 和 ``b`` 的值都应该是0.1， 但浮点型无法精确表示0.1这个值，实际上可能会存在一点偏差，由于 ``a`` 和 ``b`` 的值的计算过程不同，存在的偏差也不一样，最后两者做精确比较的结果是不相等。等学习了 :ref:`number.float` 你就知道为什么浮点型的精度有限了。如果你需要判断两个符点数是否近似相等，可以这样比较： ``if (a > b - 0.001 && a < b + 0.001) ...`` 。

.. rubric:: 习题

#. 把代码段

   .. code-block:: c
      :linenos:

      if (x > 0 && x < 10);
      else
              printf("x is out of range.\n");

   改写成下面这种形式：

   .. code-block:: c
      :linenos:

      if (____ || ____)
              printf("x is out of range.\n");

   ____应该怎么填？

#. 把代码段：

   .. code-block:: c
      :linenos:

      if (x > 0)
              printf("Test OK!\n");
      else if (x <= 0 && y > 0)
              printf("Test OK!\n");
      else
              printf("Test failed!\n");

   改写成下面这种形式：

   .. code-block:: c
      :linenos:

      if (____ && ____)
              printf("Test failed!\n");
      else
              printf("Test OK!\n");

   ____应该怎么填？

#. 有这样一段代码：

   .. code-block:: c
      :linenos:

      if (x > 1 && y != 1) {
              ...
      } else if (x < 1 && y != 1) {
              ...
      } else {
              ...
      }

   要进入最后一个else，x和y需要满足条件____ || ____。这里应该怎么填？

#. 以下哪一个 ``if`` 判断条件是多余的可以去掉？这里所谓的“多余”是指：在 ``x`` 和 ``y`` 取值的任何一种可能的情况下，如果本来应该打印 ``Test OK`` ，去掉这个多余条件后仍然打印 ``Test OK`` ，如果本来应该打印 ``Test failed`` ，去掉这个多余条件后仍然打印 ``Test failed`` 。

   .. code-block:: c
      :linenos:

      if (x<3 && y>3)
              printf("Test OK\n");
      else if (x>=3 && y>=3)
              printf("Test OK\n");
      else if (z>3 && x>=3)
              printf("Test OK\n");
      else if (z<=3 && y>=3)
              printf("Test OK\n");
      else
              printf("Test failed\n");

#. 以下两段代码是否等价？

   .. code-block:: c
      :linenos:

      if (A && B)
              statement1;
      else
              statement2;

   .. code-block:: c
      :linenos:

      if (A) {
              if (B)
                      statement1;
      } else
              statement2;

.. _cond.switch:

switch语句
------------------

``switch`` 语句可以产生具有多个分支的控制流程。它的格式是::

   switch (控制表达式) {
   case 整型常量表达式： 零或多条语句
   case 整型常量表达式： 零或多条语句
   ...
   default： 零或多条语句
   }

例如以下程序根据传入的参数1~7分别打印Monday~Sunday：

.. _cond.switch1:

.. figure:: ../images/cond.switch1.png

   switch语句

如果传入的参数是2，则从 ``case 2`` 分支开始执行，先是打印相应的信息，然后遇到 ``break`` 语句，它的作用是跳出整个 ``switch`` 语句块。C语言规定各 ``case`` 分支的常量表达式必须互不相同，如果控制表达式不等于任何一个常量表达式，则从 ``default`` 分支开始执行，通常的习惯是把 ``default`` 分支写在最后。

使用 ``switch`` 语句要注意几点：

.. index:: Fall Through

#. ``case`` 后面跟的表达式必须是常量表达式，它的值和全局变量的初始值一样必须能在编译时计算出来。
#. 在上一节讲过浮点型不适合做精确比较，所以C语言规定case后面跟的必须是整型常量表达式。
#. 进入 ``case`` 后如果没有遇到 ``break`` 语句就会一直往下执行（这称为Fall Through），后面其他 ``case`` 或 ``default`` 分支的语句也会被执行到，直到遇到 ``break`` ，或者执行到整个 ``switch`` 语句块的末尾。通常每个 ``case`` 后面都要加上 ``break`` 语句，但有时会故意不加 ``break`` ，例如：

.. _cond.switch2:

.. figure:: ../images/cond.switch2.png

   缺break的switch语句

``switch`` 语句不是必不可缺的，显然可以用一组 ``if ... else if ... else if ... else ...`` 代替，但还是推荐写程序时多用 ``switch`` ：一方面用 ``switch`` 语句会使代码更清晰，另一方面，有时候编译器会对 ``switch`` 语句做整体优化，使它比等价的 ``if/else`` 语句所生成的指令执行效率更高。
