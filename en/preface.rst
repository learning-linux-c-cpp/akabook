Preface
*************

Origin
=========

This book is originated from a vocational training course when I once worked for a training company called AKA Education (http://www.akaedu.org). One of their courses is called Linux system developer. That is a four-month fulltime training course. At the beginning students may know nothing at all about programming or Linux system, but at the end of the course they should gain very solid C programming skills and also be competent as a Linux system administrator, furthermore, they should have a deep insight about computer architecture and the functioning of operating system and device drivers.

As I see it, vocational training in computer science is more challenging than university education. Students' backgrounds vary strikingly:

* graduates, undergraduates and even high school students
* majored in computer science, other engineering disciplines and even accounting or human resource
* some are merely at the age of 20, while some are more than 35

It is all these students with different backgrounds and perception levels that constitute our class. Yet they do have common ground: to learn programming skills and have their occupation diverted to IT industry. This unique teaching experience gives rise to the birth of this book - because I couldn't find a textbook out of the shelf to utilize, I decided to write one myself.

To be a system developer is no easy job. It takes four years to graduate from university, and students majored in computer science should learn loads of fundamental courses such as calculus, linear algebra, probability, discrete mathematics, combinatorics, automata theory, compiler, computer architecture, operating system, plus loads of applied courses such as C, C++, Java, database, computer networks, information security, software engineering, computer graphics, so on and so forth. By contrast, we only have four months to train a student from scratch, and when the course is over we must assure the student can find a job as a system developer.

Why four months? That specific timespan is determined by the market. Most students won't be willing to pay for a longer and more expensive training course. We could have cut the course even shorter if we were satisfied with just teaching some Linux commands and service configurations to yield a system administrator, or teaching some JavaScript/CSS/HTML to yield a web designer. But we couldn't stop there. We aim to make students not only know WHAT and HOW, but also know WHY.

So this is four years of university education vs. four months of vocational training to achieve about the same goal, i.e. to make students capable of landing a job. What we have done is optimizing the knowledge hierarchy, so that one doesn't have to learn all those mathematics or computer science fundamentals before learning how to program. We teach Linux C programming the whole time, but we do ensure that relevant fundamental knowledge will be mentioned at every right moment. To put it another way, Linux C programming is like the string, whilst mathematics, compiler, computer architecture, operating system, computer networks, software engineering, etc, are all beads, therefore we carry forward the training course by stringing the beads. That also accounts for why we teach C programming instead of some popular programming languages like Java, Python, PHP or whatever. C is more fundamental than Java, Python or PHP because the interpreters of those languages are all written in C. Java, Python, PHP or something like those are designed for the purpose of establishing a more abstract easy-to-use programming model and hiding some aspects of computer architecture and operating system, which means they are not suitable to be the string for beads.

.. crossreference for interpreter above

Our training course is comprised of five sections:

#. Some basic Linux commands and operations. To be prepared for the next section, students will learn to do basic file and disk operations, work with an text editor and run a compiler.
#. C programming. In this section students will gain a thorough understanding of C language syntax as well as the functioning of C compiler and linker.
#. Linux system programming with C, which involves file and I/O, process, thread, network programming, information security, etc. In this section, students will learn more about what functions or services the Linux operating system provides and how to invoke them. Students will also learn some more advanced Linux system administration commands and service configurations, and will be capable of understanding or inferring how those commands and services work.
#. C++ fundamentals and object-oriented programming. With solid C programming skills, it's natural to make the transition to C++. Besides, having written lots of codes for all the topics in section 3, now students can appreciate why object-oriented programming is necessary, and how to improve their code in object-oriented paradigm.
#. Domain-specific developing, such as GUI, database, web backend, etc. Grounded by all 4 sections above, plus skills in any one of these domains, a really competent system developer is cranked out.

This book is written for the 2nd, 3rd, 4th and 5th sections as a textbook. Now you know where it comes from.

What I Care Most in Writing This Book
============================================

First of all, I strive to make it concise and precise. I won't employ jokes or other nonsense, and I will try hard to avoid ambiguities. As a programmer, I've got tons of technical books to read which costs me a lot of time. I have to go through each book quickly and spot the answer to my programming problem. So I can't read such books for fun and appreciate those unnourishing jokes, which to me is just beating around the bush. So I know what it's like when you read my book. Maybe I have little sense of humor, but I am trying to save your time.

Secondly, I try to resolve all interdependencies among those knowledge and concepts. We often find in technical books that concept A depends on concept B, which means the defintion of A can only be described in terms of the definition of B, then concept B depends on concept C, then concept C, unfortunately, depends on concept A. So which one should be introduced first? Sometimes this happens because the arrangement of topics is less than ideal. In other cases such interdependency may be inherent and unavoidable. In this book I try hard to arrange topics and employ cross-references judiciously and try not to frustrate readers when interdependency is encountered. Thererfore all you need to do is reading this booking linearly from cover to cover. If you encouter some topics that you have been quite familiar with, please at lease skim for a while to avoid omitting some key concepts depended by later chapters.

Lastly I hope this book can serve as an introductory guide to software technology. When being asked how to learn programming, many senior programmers would like to recommend a book list full of classical books. IMHO this creates more problems than it solves for newbies:

* Okay, these are all classical books, but which one should I pick up first?
* Every single book assumes I already have some sort of knowledge which I don't have. Is there a proper order to go through these classics?

No there isn't a proper order. Because these classical books are written independently, the dependencies among them are meshy rather than linear. In this book I assume no prior knowledge first and bring you along the way. Each time I will introduce to you a classical book only when I think you already have the prior knowledge to follow it. After you finish this book, you know for yourself where to go further.

What Should Be Prepared before Reading This Book
========================================================

This book aims to teach programming from scratch, so you don't need to have prior knowledge about programming. But before reading this book you should meet the following prerequisites:

#. Be familiar with basic Linux system operations, such as file and directory operation commands, text editor and software package management.
#. At least have high school mathematical knowledge to be able to follow this book. It goes without saying that the more mathematical knowledge the better, because computer technology is all about mathematics.
#. Be eager to find out how programs and computers work under the hood. Knowing WHAT and HOW is sufficient for landing a job in IT industry, but if you are equivalently eager to know WHY, then this book is for you.

Why This Book Instead of K&R
===================================

[K&R]_ is the C programming bible. All C programmers know it, and many of them recommend it to newbies. We had once used [K&R]_ as the textbook in our teaching practice, then we found it too difficult for beginners. It's nothing short of a nightmare and frustrates beginners badly. [K&R]_ is only suitable for a skillful C programmer who wants to go over his knowledge of C and tries to correct some error notions he has held about C. And I swear it's not for beginners, so don't try this at home.

Here's another thing you need to know. Before any ANSI C standard was formulated, the first edition of [K&R]_ was the de facto C standard. Then after C89 standard was formulated, [K&R]_ was revised to its second edition to comply with C89. Unfortunately, since then [K&R]_ hasn't been revised any more to reflect new amendments to C89 and the latest C99 standard, while this book complies with C99 standard and is more current than [K&R]_.

Why Learning C Programming under Linux Instead of Windows
==================================================================

C is a low level language, which means programs written in C interacts closely with operating system. And Linux is a completely open source operating system. When learning C programming under Linux, we can drill any function call down to the source code of the C runtime library, the kernel or even the hardware instructions. We can always find out implementation details to any extent that we like to. Or if we are not skillful enough, we can always find answers in all kinds of online maillists, newsgroups and IRC channels.

This is not the case under Windows, where we can't see the source code of operating system and can only rely on those inevitably ambiguous documents. Even though there are also thousands of Microsoft specialists to consult, they are only more specialized in guessing the implementation details. Under Windows you can know WHAT and HOW, but you can never know exactly WHY.

Software development under Windows often depends heavily on IDE (Integrated Development Environment) such as Visual Studio and Eclipse, which might not be good for beginners. We should start learning C programming from the most basic concepts such as compiler, linker, Makefile and command line environment. IDE conceals all these concepts with buttons and menus. Without knowledge of the underlying mechanisms you can't fully understand C programming language. Therefore IDE doesn't facilitate learning, in fact it impedes learning. Some day when you thoroughly understand how IDE integrates compiler, linker, Makefile etc you may choose to use IDE. But I guess by then you would prefer vi or Emacs instead, which are more versatile than IDE.

Conventions
==================

The font like ``The quick brown fox jumps over the lazy dog`` is used in this book for source code and terminal output. For monospace font I prefer Dejavu Sans Mono to Courier New, because digit ``1`` and lowercase letter ``l`` can be distinguished clearly, which is also the case with digit ``0`` and uppercase letter ``O``. At class I frequently find beginners copying the wrong characters printed in fonts other than Dejavu Sans Mono.

Unfortunately I can't guarantee the source codes of online version are displayed in Dejavu San Mono. That depends on readers' browser.

A source code block is like this:

.. code-block:: bash
   :linenos:

   #! /bin/sh
   VAR=1
   VAR=$(($VAR+1))
   echo $VAR

The terminal output includes Shell prompt, commands input, and results output::

   $ /bin/sh script.sh
   2

In this book we assume $ as Shell prompt.

**Words in boldface** stand for emphasis.

Acknowledgements
=======================

This book could never have come into existence without my working experience in AKA Education and without approval from the company. So special thanks go to my former leaders, Teacher Li Ming and Teacher He Jiasheng.

Besides, thanks to all the teachers that have helped me and inspired me: Di haixia, Lang Tieshan, Zhu Zhongtao, Liao Wenjiang, Han Chao, Qin Wei, Wu Wei, Zhang Di, Xing Wenpeng, He Xiaolong, Lin Xiaozhu, Wei Jianfan, Guo Tongbin, Wang Bo, Wang Lei, Hong Feng.

Thanks Broadview of PHEI for publishing the Chinese edition of this book.

Thanks to all the students and cyber friends who give valuable comments on this book.

To readers of this book, thank you for trusting me and choosing this book as your very first one to learn programming. Hope this book can live up to your expectation.
