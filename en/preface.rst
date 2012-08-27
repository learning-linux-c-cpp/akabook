Preface
*************

Origin
=========

This book is originated from a vocational training course when I once worked for a training company called AKA Education ( http://www.akaedu.org ). One of their courses is called Linux system developer. That is a four-month fulltime training course. At the beginning students may know nothing at all about programming or Linux system, but at the end of the course they should gain very solid C programming skills and also be competent as a Linux system administrator, furthermore, they should have a deep insight about computer architecture and the functioning of operating system and device drivers.

As I see it, vocational training in computer science is more challenging than college education. Students' backgrounds vary strikingly: graduates, undergraduates and even high school graduates; majored in computer science, other engineering disciplines and even accounting or human resource; some are merely at the age of 20, while some are more than 35. It is all these students with different backgrounds and perception levels that constitute our class. Yet they do have common ground: to learn programming skills and have their occupation diverted to IT industry. This unique teaching experience gives rise to the birth of this book.

To be a system developer is no easy job. It takes four years to graduate from a college, and students majored in computer science should learn loads of fundamental courses such as calculus, linear algebra, probability, discrete mathematics, combinatorics, automata theory, compiler, computer architecture, operating system, plus loads of applied courses such as C, C++, Java, database, computer networks, information security, software engineering, computer graphics, so on and so forth. By contrast, we only have four months to train a student from scratch, and when the course is over we must assure the student can find a job as a system developer. Why four months? That specific timespan is determined by the market. Most students won't be willing to pay for a longer and more expensive training course. We could have cut the course even shorter if we were satisfied with just teaching some Linux commands and service configurations to yield a system administrator, or teaching some Javascript/css/html to yield a web designer. But we couldn't stop there. We aim to make students not only know WHAT and HOW, but also know WHY.

So this is four years of college education vs. four months of vocational training to achieve about the same goal, i.e. to have students capable of landing a job. What we have done is optimizing the knowledge hierarchy, so that one doesn't have to learn all those mathematics or computer science fundamentals before learning how to program. We teach Linux C programming the whole time, but we do ensure that relevant fundamental knowledge will be mentioned at every right moment. To put it another way, Linux C programming is like the string, whilst mathematics, compiler, computer architecture, operating system, computer networks, software engineering, etc, are all beads, therefore we carry forward the training course by stringing the beads. That also accounts for why we teach C programming instead of some popular programming languages like Java, Python, PHP or whatever. C is more fundamental than Java, Python or PHP because the interpreters of those languages are all written in C. Java, Python, PHP or something like them are designed for the purpose of establishing a more abstract easy-to-use programming model and hiding some aspects of computer architecture and operating system, which means they are not suitable to be the string for beads.

.. crossreference for interpreter above

Our training course is comprised of five sections:

#. Some basic Linux commands and operations. To be prepared for the next section, students will learn to do basic file and disk operations, work with an text editor and run a compiler.
#. C programming. In this section students will gain a thorough understanding of C language syntax as well as the functioning of C compiler and linker.
#. Linux system programming with C, which involves file and I/O, process, thread, network programming, information security, etc. In this section, students will learn more about what functions or services the Linux operating system provides and how to invoke them. Students will also learn some more advanced Linux system administration commands and service configurations, and will be capable of understanding or assuming how those commands and services work.
#. C++ fundamentals and object-oriented programming. With solid C programming skills, it's natural to make the transition to C++. Besides, having written lots of codes for all the topics in section 3, now students can appreciate why object-oriented programming is necessary, and how to improve their code in object-oriented paradigm.
#. Domain-specific developing, such as GUI, database, web backend, etc. Grounded by all 4 sections above, plus skills in any one of these domains, a really competent system developer is cranked out.

This book is written for the second section as a textbook. Now you know where it comes from.

What I care most in writing this book
============================================

First of all, I strive to make it concise and precise. I won't employ jokes or other nonsense, and I will try hard to avoid ambiguities. As a programmer, I've got a lot of technical books to read which costs me a lot of time. I've got to learn the technique quickly and find answers to my programming problem. So I can't read such a book for fun and appreciate those unnourishing jokes, which to me is just beating around the bush. So I know what it's like when you read my book. Maybe I have little sense of humor, but I am trying to save your time.

Secondly, I try to resolve all interdependencies among those knowledge and concepts. We often find in technical books that concept A depends on concept B, which means the defintion of A can only be described in terms of the definition of B, then concept B depends on concept C, then concept C, unfortunately, depends on concept A. So which one should be introduced first? Sometimes this happens because the arrangement of topics is less than perfect. In another occasion such interdependency may be inherent and unavoidable. In this book I try hard to arrange topics and employ cross-references judiciously and try not to frustrate readers when interdependency is encountered.

What should be prepared before reading this book
========================================================



Why this book instead of K&R
===================================

K&R stands for the classical book The C Programming Language written by Brian W. Kernighan and Dennis Ritchie. All C programmers know it, and many C programmers recommend it to newbies. We had once employed K&R as the textbook in our teaching practice, but we soon found it too difficult for beginners. It's nothing short of a nightmare and will frustrate beginners badly. K&R is only suitable for a skillful C programmer who wants to go over his knowledge of C and tries to correct some error notions he has held about C.
