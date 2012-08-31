Basics
=========

C++ evolves from C, incorporating software engineering practice and borrowing object-oriented methodology from other languages, while keeping fully compatible with the syntax of C. In other words, a C program can definitely be compiled by a C++ compiler, although it misses all the new features of C++. This tutorial assumes audiences have solid knowledge of Linux C programming. Then you can easily upgrade your knowledge to C++. This tutorial illustrates what new features are introduced to C, why they are invented, and how to use them the right way. Although it's quick and short, it covers all the most frequently used features of C++.

Not only does C++ share the same basic syntax with C, its underlying mechanism, such as storage class, dynamic allocation, calling convention, object file format, linking convention etc., is also compatible with C. A library compiled from C code can be linked with C++ code, so that C++ code can invoke library functions written in C. In fact, some C++ compilers just translate C++ code to equivalent C code or C-like intermediate code, leaving all the rest work to a C compiler. This tutorial will address both syntax perspective and underlying perspective of C++. In order to fully understand C++, you have to know what's happening under the hood.

C functions in the same subsystem often have names with the same prefix. For example, ''sigaction'', ''sigsuspend'', ''sigprocmask'', ''sigaddset'', ''sigismember'' etc. all belong to the signal subsystem. ''pthread_create'', ''pthread_exit'', ''pthread_join'' etc. all belong to the phread subsystem. That common prefix indicates a **namespace**. Besides, ''pthread_'' has sub-namespace: ''pthread_mutex_init'', ''pthread_mutex_lock'' etc. all belong to the ''pthread_mutex_'' sub-namespace. If those functions were inplemented in C++, they would have been called ''pthread::create'', ''pthread::exit'', ''phtread::mutex::init'', ''phtread::mutex::lock'' etc. - double colon concatenation is the way to define namespaces and sub-namespaces in C++. Not only functions can be named under a certain namespace, global variable and type names can also be prefixed.

namespace
-------------

Here comes the C++ hello world:

<code cpp>
#include <iostream> 
 
int main() 
{ 
	std::cout<<"Hello world!"<<std::endl;
	return 0;
}
</code>

C++ source filename usually has a suffix of ''.cpp'', ''.cc'', or ''.cxx''. On Linux you can compile a C++ program with ''g++'' command. You probably need to install a package called ''g++''. Its command line options are almost the same as ''gcc''.

  $ g++ main.cpp 
  $ ./a.out 
  Hello world!

<note>
Don't name a C++ source file with the suffix ''.c''. It will confuse the ''g++'' compiler.
</note>

Now let's check the source code. ''iostream'' is a header file from C++ standard library. According to C++ convention, header files from C++ standard library will not have a suffix ''.h'', while a user header file should have. On some Linux distributions those standard header files can be found under ''/usr/include/c++''. ''std::cout'' and ''std::endl'' are declared in this header file. This line is equivalent to the following C code.

<code cpp>
printf("Hello world!\n");
</code>

''std::cout'' stands for standard output, while ''std::endl'' stands for a new line character. We see things from C++ standard library are under the ''std::'' namespace. The ''<<'' operator used to be the bitwise left shift operator. Here it is redefined as an output operator. We can concatenate all the items we want to output to ''std::cout'' by ''<<'' operator. How this is achieved will be explained in later sections.

Whenever we use a name under some namespace, we should write the prefix, even in a clear context. That's very tedious. And that's the way we write C codes all the time. But with the following syntax in C++ you can save some typing.

<code cpp>
#include <iostream>
using namespace std;
 
int main() 
{ 
	cout<<"Hello world!"<<endl;
	return 0;
}
</code>

After the declaration ''using namespace std;'', names like ''cout'' and ''endl'' will automatically be searched under ''std::'', if they are not found in other scope like local or global scope. ''cout'' is not another thing. It's the same thing as ''std::count'', just to be named shortly. It can be a little confusing if there is also a global variable named ''cout'', in which case we can use ''::cout'' instead to address the name ''cout'' from global scope rather than ''std::''.

Bool Type
-------------

C++ introduced a new primitive type: bool type. You can declare a variable of bool type. And you can use two new bool constants ''true'' and ''false''. Bool value is often used in the control expression of a branch or loop.

<code cpp>
#include <iostream> 
using namespace std; 
 
int main(void) 
{ 
	bool cond = true; 
	if (cond) 
	{ 
		cout << "The condition is true." << endl; 
	}
	return 0;
}
</code>

If bool value is used in an arithmetic expression, it will be coverted to 1 or 0 of appropriate type.

Reference Type
-----------------

C++ introduced the new reference type. The difference between C pointer and C++ reference has always been confusing to newbies. In the following code, ''n'' is a reference, while ''m'' is the referent referred to by ''n''.

<code cpp>
int m;
int &n = m;
</code>

From now on, ''n'' is an alias to ''m''. ''n'' is neither a copy of nor a pointer to ''m''. ''n'' is exactly ''m'' itself, with a different name. If you assign 3 to ''n'', you actually assign that number to ''m''. Therefore, reference type is subject to the following constraint.

  *A reference must be initialized when it is defined. You cannot define a reference without initialization first and assign to it later.
  *A pointer can be ''NULL'', which points to nothing. However, a reference must refer to some object. There's no ''NULL'' reference.
  *A pointer can first point to ''a'' and then point to ''b''. However, once a reference is defined and initialized, it cannot be assigned to another object. It is fixed to be the alias to a particular object through out its lifetime.

Reference type facilitates function argument passing. For example, the following ''myswap'' function with parameters of reference type can easily swap its arguments.

<code cpp>
#include <iostream>
using namespace std;

void myswap(int &x, int &y)
{
	int temp;
	temp = x;
	x = y;
	y = temp;
}

int main(void)
{
	int i=3, j=4;
	myswap(i, j);
	cout<<"i="<<i<<endl<<"j="<<j<<endl;
}
</code>

Think about how we achieve this in C. We declare ''myswap'' function to take two pointer parameters, which points to the arguments. Then we use pointer dereference to access and swap those arguments. With reference type, the code is simplified, but the concept behind is a little confusing. We know that function parameters should be allocated on the stack frame of that function. If ''x'' and ''y'' are aliases to ''i'' and ''j'' in ''main'' function, should ''myswap'' function refer to objects on ''main'' fuction's stack frame? Well, definitely not. I said ''x'' is an alias to ''i'' just from the syntax perspective. From the underlying perspective, reference can be implemented as a const pointer, which points to a particular object and cannot be changed. The code above can be recognized by a C++ compiler as the following C code:

<code cpp>
void myswap(int *const x, int *const y)
{
	int temp;
	temp = *x;
	*x = *y;
	*y = temp;
}
...
myswap(&i, &j);
...
</code>

The rationale behind reference type is more than code simplicity. The main reason is security. Pointer is too versatile to be used safely. If I pass the address of my variable ''a'' to your pointer ''p'', I want you to manipulate exactly that variable through ''*p'' or ''p[0]''. I certainly don't want you to access ''*(p+1)'' or ''p[1]''. However, once I passed the address to you, it's at your disposition. I have no way to keep you from touching ''p[1]'', whether purposely or inadvertently. In this scenario, reference type comes into play. If you can just keep a reference to my variable, I will feel much safer.

Initialization
-----------------


C++ introduced another form of initialization. For built-in types such as ''int'', ''char'', ''float'', the following two forms are equivalent.

<code cpp>
int i = 10;
int i(10);
</code>

The first form is the good old C style. The second function-like form is the new C++ style. For class instances, initializing by the function-like form means invoking its constructor, which can be passed more than one arguments. That will be further detailed in later sections.

new and delete Operators
-------------------------------

''new'' and ''delete'' are two new operators for allocating and deallocating memory. While in C we use library functions ''malloc'' and ''free'', in C++ we use these two operators.

<code cpp>
#include <iostream> 
using namespace std; 

int main(void) 
{ 
	/* int *arraysize=(int *)malloc(sizeof(int)); */	
	int *arraysize = new int;
	
	cin >> *arraysize;

	/* int *array = (int *)malloc((*arraysize)*sizeof(int)); */
	int *array = new int[*arraysize];

	for(int i = 0; i < *arraysize; i++)  
		array[i] = i;
 
	for(int i = 0; i < *arraysize; i++) 
		cout << array[i] << ","; 

	cout << endl;
	/* free(array); */
	delete[] array;
	/* free(arraysize); */
	delete arraysize;
	return 0;
}
</code>

Here ''cin >>'' behaves like ''scanf'', reading from standard input into variables. Codes in comments are alternative C codes for the same purpose. Note to deallocate an array of objects, we should use ''delete[]''. If those objects are class instances, ''[]'' is required in ''delete[]''. For built-in types, however, ''[]'' can be omitted. For consistency, we should always use ''delete[]'' in such cases.

Think for a minute: what's the difference between ''int *a = new int[10];'' and ''int *a = new int(10);''?

<note>
Never ''delete'' an area allocated by ''malloc'', or ''free'' an area allocated by ''new''. C++ operators and C library functions have different heap management mechanisms, so they cannot be intermixed.
</note>

default arguments of functions
------------------------------------

When declaring a function in C++, we can give default values to some parameters. That way when we call this function later, we need not specify values for these parameters if the default behavior is just what we want. Function invocation can be more concise with these arguments omitted. Default parameters can be assigned in function prototypes or definitions, but we can assign only once. The best practice is to assign in prototypes. For example,

<code cpp>
void foo(int x=0, int y=0);
void foo(int x, int y)
{
…
}
</code>

If a function has more than one parameters with default values, we can take default values only from the rightmost parameter to the left. The above function can be invoked as:

<code cpp>
foo(); //x=0, y=0
foo(1); //x=1, y=0
foo(1,2); //x=1, y=2
</code>

If ''foo'' has two parameters and only one of them has a default value, it can be declared as

<code cpp>
void foo(int x, int y=0);
</code>

but cannot be declared as

<code cpp>
void foo(int x=0, int y);
</code>

because invocation like ''foo(,2);'' is illegal.

overloaded functions
-------------------------

Let's begin with an example first.

<code cpp>
#include <iostream> 
using namespace std;

int add(int a, int b) 
{ 
	return a+b; 
} 
 
double add(double a, double b)
{ 
	return a+b; 
}

int main(void) 
{
	cout<<add(1,2)<<endl<<add(2.1,3.14)<<endl;
	return 0;
}
</code>

In this code snippet we define two functions with the same name ''add''. One does addition of type ''int'', and the other does addition of type ''double''. This is allowed in C++ and named overloaded funtion. With this rule, functions with similiar behavior and interface can share the same name. It also makes function invocation more concise and uniform. When an overloaded function is invoked, the compiler chooses the most conforming definition according to the types of arguments passed. Therefore, all overloaded function definitions must have different numbers or types of parameters.

Think for a minute: if two function definitions are different only in return value types, can they be overloaded?

Now take an exercise: in the example above, if we replace all ''double'' with ''float'', does it still work? Why?

From the syntax perspective, the compiler can differentiate overloaded functions by numbers and types of parameters. But from the underlying perspective, how can two functions with the same name be linked into one object file without ambiguity? We know the compiler cares about types, while the linker only cares about names. Definition and invocation of the same name will be linked together. Now we have two defintions, then which one will be linked to a particular invocation? The answer is: compiler won't let linker see definitions with the same name. C++ compiler has a processing step called name mangling, renaming overloaded functions to different names, so that linker won't be confused.

If we compile the example above to an object file, and explore the symbol table with ''nm'' utility, we see the two ''add'' functions have different link names.

  $ g++ main.cpp
  $ nm a.out
  ...
  08048670 T _Z3adddd
  08048664 T _Z3addii
  ...

These link names are fabricated with the magic number (''_Z3''), the function name (''add'') and the parameter type signature (''dd'' and ''ii'').

overloaded operators
--------------------------

Look at this example first:

<code cpp>
#include <iostream>
using namespace std;

struct Complex {
       double real, img;
};

Complex Add(const Complex &a, const Complex &b)
{
	Complex temp = { a.real + b.real, a.img + b.img };
	return temp;
}

int main(void)
{
	Complex a = { 1.1, 2.2 };
	Complex b = { 3.3, 4.4 };
	Complex c = Add(a, b);
	cout << c.real << '+' << c.img << 'i' << endl;

	return 0;
}
</code>

Note in C ''struct Complex'' can be used as a type name while in C++ the tag ''Complex'' itself is sufficient to be a type name. C++ also supports the C style for compatibility.

In C++, functions can be named with an operator and the keyword ''operator''. This is called overloaded operator. The ''Add'' function above can be rewritten as

<code cpp>
Complex operator+(const Complex &a, const Complex &b)
{
	Complex temp = { a.real + b.real, a.img + b.img };
	return temp;
}
</code>

Then the invocation can be written as

<code cpp>
Complex c = a + b;
</code>

It looks more consistent: we can add two operands with ''+'' operator, whether they are of built-in type or user-defined type. When a normal function is invoked, the function name precedes its arguments. But when an overloaded binary operator such as ''+'' is invoked, the operator appears between its two arguments, which is called infix form.

Think for a minute: the ''operator+'' function takes two parameters of reference type, but its return type is not a reference, why?

Since operator is a special form of function, overloaded operators are in essence overloaded functions. Note that overloading doesn't change precedence and associativity of operators. An overloaded ''*'' still has higher precedence over an overloaded ''+''. Like overloaded function, when the compiler encounters an expression, it makes the decision about which operator definition to choose based on the types of operands. However, we cannot redefine the behavior of an expression of built-in types. For example, we cannot overload ''+'' operator with two arguments of built-in type ''int''.

Now we see why ''<<'' operator following ''cout'' is an output operator: because it is overloaded. We can overload it further to accommodate the new ''Complex'' type we has defined.

<code cpp>
ostream& operator<<(ostream &o, const Complex &a)
{
	o << a.real << '+' << c.img << 'i' << endl;
	return o;
}
</code>

Then we can output a ''Complex'' object directly to ''cout''.

<code cpp>
Complex c = a + b;
cout << c;
</code>

''cout'' is of type ''std::ostream''. Here ''cout'' is the left operand of ''<<'' and a ''Complex'' object is the right operand, so our overload operator matches. Note the overloaded operator returns a reference to an ''ostream'' object which is ''cout'' in fact. That means the value of expression ''cout << c'' is just ''cout''. That's why we can concatenate all things we want to output one after another with several ''<<'' operators. We can also do this:

<code cpp>
cout << a << '+' << b << '=' << c << endl;
</code>

We can freely intermix and chain objects of built-in types and user-defined types between ''<<'' operators. The compiler tries to find the right function to call.

inline functions
----------------------
We know that macros in C can take parameters and look like functions. For example,

<code cpp>
#define DIV(x,y) ((x)/(y))
</code>

But these macros should be carefully used. If the parentheses above are omitted, the semantics may be different.

C++ introduced a new macro definition syntax called inline function. In fact, C99, the latest C standard, borrows this syntax from C++. By prefixing a function definition with keyword ''inline'', the compiler will treat an invocation as function-like macro and expand it when the optimization switch is on. Otherwise, the compiler will treat it like a plain function call. For example,

<code cpp>
inline void DIV(int x, int y)
{
	return x / y;
}
</code>

This inline function definition behaves like the macro definition above when compiled with ''g++'' options like ''-O'' ''-O1'' ''-O2'' ''-O3''. If compiled with no optimization, this function is just a normal function.

Inline function makes the best of both worlds: it has the efficiency as macros and the safety as functions. The compiler will do type checking and conversion before expanding an inline function, assuring its safety. Besides, we need not worry about whether the body is correctly parenthesized.

Note the keyword ''inline'' should be with function definition rather than declaration.

<code cpp>
void foo(int x, int y);
inline void foo(int x, int y)
{
…
}
</cpp>

Like macro definition, inline function definition can be put directly into a header file, eliminating the need to write a declaration.


calling C library functions from C++ code
-----------------------------------------------

Object files generated by C compiler can be linked with C++ code, and C functions can be directly invoked from C++ code. But how can this work?

We know that C++ has a mechanism called name mangling. All functions in C++ code have mangled link names after compilation. If we implement a function ''int add(int, int)'' in one C++ code and call that function in another C++ code:

<code cpp>
// add.cpp
int add(int a, int b)
{
	return a + b;
}
</code>

<code cpp>
// main.cpp
int add(int a, int b);
int main(void)
{
	add(3, 5);
	return 0;
}
</code>

Then object file ''add.o'' will have an implementation with link name ''_Z3addii'', and object file ''main.o'' will have an invocation with link name ''_Z3addii''. They can be linked together because they have the same mangled name.

In the case when C++ code calls a C function, things become different.

<code c>
// add.c
int add(int a, int b)
{
	return a + b;
}
</code>

The object file ''add.o'' will have an implementation with link name exactly ''add'', which cannot be linked with C++ code ''main.cpp'':

  $ gcc -c add.c
  $ g++ main.cpp add.o
  /tmp/cckyAIcG.o: In function `main':
  main.cpp:(.text+0x21): undefined reference to `add(int, int)'
  collect2: ld returned 1 exit status

We should wrap the declaration of function ''add'' with ''extern "C" {...}'' and then call it:

<code cpp>
// main.cpp
extern "C" {
int add(int a, int b);
}

int main(void)
{
	add(3, 5);
	return 0;
}
</code>

C++ compiler will know that ''add'' is a C function and generate code calling ''add'' instead of calling ''_Z3addii''.

In order to be used by C++ code, C library headers oftern have the following wrapper:

<code c>
#ifdef __cplusplus
extern "C" {
#endif

......
#ifdef __cplusplus
}
#endif
</code>

If it's included by another C code and compiled by C compiler, __cplusplus defines to nothing, therefore the ''extern "C"'' wrapper is skipped. If it's included by a C++ code and compiled by C++ compiler, however, the ''extern "C"'' wrapper comes into play, informing C++ compiler not to mangle names within the wrapper.

Besides, C++ standard library provides many header files that wrap the corresponding C header files. For example, ''stdio.h'' is wrapped in ''cstdio''. When calling C standard library functions from C++ code, we should include the wrapped version.

<code cpp>
#include <cstdio>
#include <ctime>
</code>

