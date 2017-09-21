## Chapter 2: Constructors, Destructors,and Assignment Operators

Almost every class you write will have one or more constructors, a destructor, and a copy assignment operator. Little wonder. These are
your bread-and-butter functions, the ones that control the fundamental
operations of bringing a new object into existence and making sure
it’s initialized, getting rid of an object and making sure it’s properly cleaned up, and giving an object a new value. Making mistakes in these functions will lead to far-reaching — and unpleasant — repercussions throughout your classes, so it’s vital that you get them right.
In this chapter, I offer guidance on putting together the functions that
comprise the backbone of well-formed classes. 

[**Item 5:** ](https://sahibyar.gitbooks.io/effective-cpp-summary/content/chapter-1-accustoming-yourself-to-c-++/item-1.html) **Know what functions C++ silently writes a calls.**

[**Item 6:**](https://sahibyar.gitbooks.io/effective-cpp-summary/content/chapter-1-accustoming-yourself-to-c-++/item-2.html) **Explicitly disallow the use of compiler-generated functions you do not want.**

[**Item 7:**](https://sahibyar.gitbooks.io/effective-cpp-summary/content/chapter-1-accustoming-yourself-to-c-++/item-3.html) **Prevent exceptions from leaving destructors.**

[**Item 8:**](https://sahibyar.gitbooks.io/effective-cpp-summary/content/chapter-1-accustoming-yourself-to-c-++/item-4.html) **Prevent exceptions from leaving destructors.**

[**Item 9:**](https://sahibyar.gitbooks.io/effective-cpp-summary/content/chapter-1-accustoming-yourself-to-c-++/item-4.html) **Never call virtual functions during construction or destruction.**

[**Item 10:**](https://sahibyar.gitbooks.io/effective-cpp-summary/content/chapter-1-accustoming-yourself-to-c-++/item-4.html) <b>Have assignment operators return a reference to `*this`.</b>

[**Item 11:**](https://sahibyar.gitbooks.io/effective-cpp-summary/content/chapter-1-accustoming-yourself-to-c-++/item-4.html) <b>Handle assignment to self in `operator=`.</b>

[**Item 12:**](https://sahibyar.gitbooks.io/effective-cpp-summary/content/chapter-1-accustoming-yourself-to-c-++/item-4.html) **Copy all parts of an object.**