## Chapter 3: Resource Management

A resource is something that, once you’re done using it, you need to return to the system. If you don’t, bad things happen. In C++ programs, the most commonly used resource is dynamically allocated memory (if you allocate memory and never deallocate it, you’ve got a memory leak), but memory is only one of many resources you must manage. Other common resources include file descriptors, mutex locks, fonts and brushes in graphical user interfaces (GUIs), database connections, and network sockets. Regardless of the resource, it’s important that it be released when you’re finished with it.. 

[**Item 13:**](https://sahibyar.gitbooks.io/effective-cpp-summary/content/chapter-2-constructors-destructors-and-assignment-operators/item-5.html) **Know what functions C++ silently writes a calls.**

[**Item 6:**](https://sahibyar.gitbooks.io/effective-cpp-summary/content/chapter-2-constructors-destructors-and-assignment-operators/item-6.html) **Explicitly disallow the use of compiler-generated functions you do not want.**

[**Item 7:**](https://sahibyar.gitbooks.io/effective-cpp-summary/content/chapter-2-constructors-destructors-and-assignment-operators/item-7.html) **Prevent exceptions from leaving destructors.**

[**Item 8:**](https://sahibyar.gitbooks.io/effective-cpp-summary/content/chapter-2-constructors-destructors-and-assignment-operators/item-8.html) **Prevent exceptions from leaving destructors.**

[**Item 9:**](https://sahibyar.gitbooks.io/effective-cpp-summary/content/chapter-2-constructors-destructors-and-assignment-operators/item-9.html) **Never call virtual functions during construction or destruction.**

[**Item 10:**](https://sahibyar.gitbooks.io/effective-cpp-summary/content/chapter-2-constructors-destructors-and-assignment-operators/item-10.html) <b>Have assignment operators return a reference to `*this`.</b>

[**Item 11:**](https://sahibyar.gitbooks.io/effective-cpp-summary/content/chapter-2-constructors-destructors-and-assignment-operators/item-11.html) <b>Handle assignment to self in `operator=`.</b>

[**Item 12:**](https://sahibyar.gitbooks.io/effective-cpp-summary/content/chapter-2-constructors-destructors-and-assignment-operators/item-12.html) **Copy all parts of an object.**