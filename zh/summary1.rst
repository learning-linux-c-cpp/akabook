本阶段总结
=====================

善于学习的人都应该善于总结。本书的编排顺序充分考虑到知识的前后依赖关系，保证在讲解每个新知识点的时候都只用到前面章节讲过的知识，但正因为如此，很多相互关联的知识点被拆散到多个章节中了。我们一章一章地纵向学习过来之后，应该理出几个横切面，把拆散到各章节中的知识点串起来。请从以下几个方面整理和复习。

#. C的语法规则

   *  源文件中所有函数定义之外可以出现哪些语法元素？
   *  函数定义之中可以出现哪些语法元素？
   *  语句有哪几种？
   *  哪些语法元素需要遵循标识符的命名规则？
   *  表达式由哪些语法元素组成？
   *  到目前为止学过哪些运算符？它们的优先级和结合性是怎样的？
   *  哪些运算符取操作数的左值？哪些运算符有Side Effect？
   *  哪些运算符的操作数必须是整型？哪些运算符的操作数必须是算术类型？哪些运算符的操作数必须是标量类型？
   *  哪些表达式可以做左值？哪些表达式只能做右值？
   *  哪些地方必须用常量表达式？哪些地方必须用整型常量表达式？

#. 思维方法与编程思想

   *  组合规则， :ref:`expr.expression`
   *  Least Surprise， :ref:`func.parameter`
   *  充分条件与必要条件， :ref:`func.scope`
   *  封装， :ref:`cond.ifelse`
   *  布尔逻辑， :ref:`cond.bool`
   *  递归， :ref:`func2.recurse`
   *  函数式编程， :ref:`iter.while`
   *  迭代（ :doc:`iter` ）与增量式求解（ :ref:`sortsearch.insertion` ）
   *  抽象， :ref:`struct.dataabstraction`
   *  避免硬编码， :ref:`array.statrandom`
   *  数据驱动， :ref:`array.multidim`
   *  分而治之， :ref:`sortsearch.mergesort`
   *  折半查找， :ref:`sortsearch.binarysearch`
   *  回溯， :ref:`stackqueue.dfs`

#. 调试方法

   *  编译错误、运行时错误与语义错误， :ref:`intro.debug`
   *  增量式开发， :ref:`func2.incremental`
   *  打印语句与Scaffold， :ref:`func2.incremental`
   *  gdb， :doc:`gdb`
   *  DbC与Assertion， :ref:`sortsearch.binarysearch`
