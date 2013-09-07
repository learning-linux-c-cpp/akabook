Template and Generic Programming
=======================================

**TODO: THIS WHOLE CHAPTER NEEDS OVERHAUL AND BROADENING**

.. figure:: ../images/road-works-sign.gif

Template
-----------

C++ has many different data types:

* primitive type: ``int``, ``char``, ``float``, ``double`` and so on
* pointer
* class and struct
* array

These data types are quite different but have some similar operations. However, so far we cannot write a general function to apply to all these types. For example, we can't write a single function ``power(x, exp)`` applying to all these types: ``int``, ``float``, ``class Complex`` and so on. C++ provides templates to implement general functions like this. We can define template classes or template functions with the ``template`` keyword. This is called **generic programming**. For example,

.. code-block:: c++
   :linenos:

   template <class T, int max>
   struct Buffer
   {
           T v[max];
   };

   Buffer <int, 10> intBuf10;

   template <class T>
   T power(T a, int exp)
   {
     T ans = a;
     while (--exp > 0) {
       ans *= a;
     }
     return (ans);
   }

The parameters in ``<>`` are template parameters. Through template parameters we can pass not only values but also types, in which case we should declare the type parameter with keyword ``class`` or ``typename``. Even declared with ``class``, the type parameter can be built-in type as well as class type. You can consider template as a kind of macro. For example, if we use ``Buffer <int, 10>`` as a type, the following type definition will be generated:

.. code-block:: c++
   :linenos:

   struct Buffer
   {
           int v[10];
   };

If we invoke ``power(c, 5)`` and ``c`` is a ``Complex`` object, the following function definition will be generated;

.. code-block:: c++
   :linenos:

   Complex power(Complex a, int exp)
   {
     Complex ans = a;
     while (--exp > 0) {
       ans *= a;
     }
     return (ans);
   }

If we invoke ``power(2.13, 4)``, another function definition will be generated. This processing is similar to macro expansion. It's called **template instantiation**. Template definitions are usually placed in header files.

Inheritance is invented for code reuse, so is template. Then what makes reusable code? Suppose you wrote a function ``func1`` yesterday, and today you write code that calls ``func1``. That's the basic form of code reuse. Now Suppose you wrote a template function ``power`` yesterday to handle power calculation of built-in types, and today you implement a ``Complex`` class. It's interesting to note that the function you wrote yesterday can also apply to the type you implement today. The template function ``power`` works with any new type as long as that type implements the overloaded operator ``*=``. This case is similar to what we saw in the last section. Suppose you wrote the ``Animal`` class and the ``whosay`` function yesterday, and add a new class ``Dog`` today which inherits from the ``Animal`` class. Then the function ``whosay`` you wrote yesterday can also apply to the new derived class ``Dog``, because it implements the ``saywhat`` interface required by ``Animal`` class.

Now try it yourself: write a template function ``myswap`` to swap two objects of any type. Test your code with different kinds of objects. Note the template cannot be named ``swap`` because there is one in C++ standard library.

Template Class
-------------------

Now we implement a template class. It behaves as a stack (supports push and pop) and can hold objects of any type.

stack.h

.. code-block:: c++
   :linenos:

   #include <iostream>
   using namespace std;

   template<class T> class Node {
    public:
       Node(T invalue): m_Value(invalue), m_Next(0) {}
       ~Node() ;

       T getValue() const {return m_Value;}
       void setValue(T value) {m_Value = value;}
       Node<T>* getNext() const {return m_Next;}
       void setNext(Node<T>* next) {m_Next = next;}
    private:
       T m_Value;
       Node<T>* m_Next;
   };

   template<class T> Node<T>::~Node() {
       cout << m_Value << " deleted " << endl;
       if(m_Next) {
           delete m_Next;
       }
   }

   template<class T> class Stack {
    public:
       Stack(): m_Head(0), m_Count(0) {}
       ~Stack<T>() {delete m_Head;}
       void push(const T& t);
       T pop();
       T top() const;
       int count() const;
    private:
       Node<T> *m_Head;
       int m_Count;
   };

   template <class T> void Stack<T>::push(const T& value) {
       Node<T> *newNode = new Node<T>(value);
       newNode->setNext(m_Head);
       m_Head = newNode;
       ++m_Count;
   }

   template <class T>  T Stack<T>::pop() {
       Node<T> *popped = m_Head;
       if (m_Head != 0) {

           m_Head = m_Head->getNext();
           T retval = popped->getValue();
           popped->setNext(0);
           delete popped;
           --m_Count;
           return retval;
       }
       return 0;
   }

   template <class T> inline T Stack<T>::top() const {
       return m_Head->getValue();
   }

   template <class T> inline int Stack<T>::count() const {
       return m_Count;
   }

Since template class definition can be placed in a header file, there's no ``stack.cpp``.

main.cpp

.. code-block:: c++
   :linenos:

   #include "stack.h"

   int main(void) {
       Stack<int> intstack1, intstack2;
       int val;
       for(val = 0; val < 4; ++val) {
           intstack1.push(val);
           intstack2.push(2 * val);
       }
       while (intstack1.count()) {
           val = intstack1.pop();
           cout << val << endl;
       }
       Stack<char> stringstack;
       stringstack.push('A');
       stringstack.push('B');
       stringstack.push('C');
       char val2;
       while (stringstack.count()) {
           val2 = stringstack.pop();
           cout << val2 << endl;
       }
       cout << "Now intstack2 will be destructed." << endl;
       return 0;
   }

Here is the result::

   3 deleted 
   3
   2 deleted 
   2
   1 deleted 
   1
   0 deleted 
   0
   C deleted 
   C
   B deleted 
   B
   A deleted 
   A
   Now intstack2 will be destructed.
   6 deleted 
   4 deleted 
   2 deleted 
   0 deleted 

At the end of ``main`` function, ``intstack2`` is automatically destructed because it's allocated on the stack and runs out of duration. But the elements contained in ``intstack2`` are allocated by the ``new`` operator, how can they be destructed one after another? Please find the answer from the code.

Template classes like this are called **containers**. They can hold elements of any type and can automatically manage the allocation and deallocation of these elements. C++ standard library has many different containers, each providing different interfaces for storing and accessing elements.
