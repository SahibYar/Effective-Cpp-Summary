## Chapter 3: Resource Management

A resource is something that, once you’re done using it, you need to return to the system. If you don’t, bad things happen. In C++ programs, the most commonly used resource is dynamically allocated memory (if you allocate memory and never deallocate it, you’ve got a memory leak), but memory is only one of many resources you must manage. Other common resources include file descriptors, mutex locks, fonts and brushes in graphical user interfaces (GUIs), database connections, and network sockets. Regardless of the resource, it’s important that it be released when you’re finished with it.

[**Item 13:**](https://sahibyar.gitbooks.io/effective-cpp-summary/content/chapter-3-resource-management/item-13.html) **Use objects to manage resources.**

[**Item 14:**](https://sahibyar.gitbooks.io/effective-cpp-summary/content/chapter-3-resource-management/item-14.html) **Think carefully about copying behavior in resource-managing classes.**

[**Item 15:**](https://sahibyar.gitbooks.io/effective-cpp-summary/content/chapter-3-resource-management/item-15.html) **Provide access to raw resources in resource-managing classes.**

[**Item 16:**](https://sahibyar.gitbooks.io/effective-cpp-summary/content/chapter-3-resource-management/item-16.html) **Use the same form in corresponding uses of `new` and `delete`.**

[**Item 17:**](https://sahibyar.gitbooks.io/effective-cpp-summary/content/chapter-3-resource-management/item-17.html) **Store newed objects in smart pointers in standalone statements.**