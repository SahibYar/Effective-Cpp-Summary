### Item 13 - Use objects to manage resources.
Suppose we're working with a library for modeling investments (e.g., stocks, bonds, etc.), where the various investment types inherit from a root class `Investment`:
```C++
class Investment { ... };     // root class of hierarchy of investment types
```
Further suppose that way the library provides us with `Investment` objects is through a factory function
> A function that returns a base class pointer to a newly-created derived class object.

```C++
Investment* createInvestment();    // return ptr to dynamically allocated object in the Investment
                                  // hierarchy; the caller must delete it (parameters omitted for simplicity)
```
As the comment indicates, callers of `createInvestment` are responsible for deleting the object that function returns when they are done with it. Consider, then, a function `f` written to fulfill this obligation:
```C++
void f()
{
   Investment* pInv = createInvestment();   // call factory function
   ...                                      // use pInv
   delete pInv;                             // release object
```
This looks okay, but there are several ways `f` could fail to delete the investment object it gets from `createInvestment`. There might be a premature `return` statement somewhere inside the "..." part of the function. If such a `return` were executed, control would never reach the `delete` statement. A similar situation would arise if the uses of `createInvestment` and `delete` were in a loop, and loop was prematurely exited by a `break` or `goto` statement. Finally, some statement inside the "..." might throw an exception. If so, control would again not get to the `delete`.Regardless of how the `delete` were to be skipped, we'd leak not only the memory containing the investment object but also any resources help by that object.

Many resources are dynamically allocated on the heap, are used only within a single block or function, and should be released when control leaves that block or function. The standard library's `auto_ptr` is tailor-made for this kind of situation. `auto_ptr` is a pointer-like object (a _smart pointer_) whose destructor automatically calls `delete` on what it points to. Here's how to use `auto_ptr` to prevent `f`'s potential resource leak:
```C++
void f()
{
   std::auto_ptr<Investment> pInv(createInvestment());   // call factory function
   ...                // use pInv as before
}                     // automatically delete pInv via auto_ptr's dtor
``` 
This simple example demonstrates the two critical aspects of using objects to manage resources:
* **Resources are acquired and immediately turned over to resource-managing objects.** Above, the resource returned by `createInvestment` is used to initialize the `auto_ptr` that will manage it. In fact, the idea of using objects to manage resources is often called _Resource Acquisition Is Initialization (RAII)_, because it's so common to acquire a resource and initialize a resource-managing object the same statement. Sometimes acquired resources are _assigned_ to resource-managing objects instead of initializing them, but either way, every resource is immediately turned over to a resource-managing object at the time the resource is acquired.
* **Resource-managing objects use their destructors to ensure that resources are released.** Because destructors are called automatically when an object is destroyed (e.g., when an object goes out of scope), resources are correctly released, regardless of how control leaves a block.

Because an `auto_ptr` automatically deletes what it points to when `auto_ptr` is destroyed, it's important that there never be more than one `auto_ptr` pointing to an object. If there were, the object would be deleted more than once, and that would put your program on the fast tract to undefined behavior. To prevent such problems, `auto_ptr` have an unusual characteristic: copying them (via copy constructor or copy assignment operator) sets them to `null` and the copying pointer assumes sole ownership of the resource!
```C++
// pInv1 points to the object returned from createInvestment
std::auto_ptr<Investment> pInv1 (createInvestment());

// pInv2 now points to the object; pInv1 is now null
std::auto_ptr<Investment> pInv2(pInv1);

// now pInv1 points to the object; and pInv2 is null
pInv1 = pInv2;
```
An alternative to `auto_ptr` is a _reference-counting smart pointer (RCSP)._ An RCSP is a smart pointer that keeps track of how many objects point to a particular resource and automatically deletes the resource when nobody is pointing to it any longer. As such, RCSPs offer behavior that is similar to that of garbage collection. Unlike garbage collection, however, RCSPs can't break cycles of references (e.g., two otherwise unused objects that point to one another).








