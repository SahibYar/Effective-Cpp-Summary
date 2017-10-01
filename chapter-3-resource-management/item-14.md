### Item 14 - Think carefully about copying behavior in resource-managing classes.
Suppose you're using a C API to manipulate mutex objects of type `Mutex` offering functions `lock` and `unlock`:
```C++
void lock(Mutex* pm);        // lock mutex pointed to by pm
void unlock(Mutex* pm);      // unlock the mutex
```
To make sure that you never forget to unlock a `Mutex` you've locked, you'd like to create a class to manage locks. The basic structure of such a class is dictated by the RAII principle that resources are acquired during construction and released during destruction:
```C++
class Lock {
public:
    explicit Lock(Mutex* pm): mutexPtr(pm)
    { lock(mutexPtr); }        // acquire resource
    ~Lock(){ unlock(mutexPtr); }    //release resource
private:
    Mutex* mutexPtr;
};
```
Clients use `Lock` in the conventional RAII fashion:
```C++
Mutex m;            // define the mutex you need to use
...
{                   // create block to define critical section
    Lock ml(&m);    // lock the mutex
    ...             // perform critical section operations
}                   //  automatically unlock mutex at end of block
```
This is fine, but what would happen if a `LOck` object is copied?
```C++
Lock ml1(&m);        // lock m
Lock ml2(ml1);       // copy ml1 and to ml2 — what would happen here ?
```
Most of the time, you'll want to choose one of the following possibilities:
* **Prohibit copying.** In many cases, it makes no sense to allow RAII objects to be copied. This is likely to be true for a class like `Lock`, because it rarely makes sense to have "copies" of synchronization primitives. When copying makes no sense for an RAII class, you should prohibit it. [Item 6](https://sahibyar.gitbooks.io/effective-cpp-summary/content/chapter-2-constructors-destructors-and-assignment-operators/item-6.html) explains how to do that: declare the copying operations private. For `Lock`, that could look like this:
```C++
class Lock: private Uncopyable {    // prohibiting copying
public:
    ...                            // as before
}
```

* **Reference-count the underlying resource.** Sometimes it's desirable to hold on to a resource until the last object using it has been destroyed. When that's the case, copying an RAII object should increment the count of the number of objects referring to the resource. This is the meaning of "copy" used by `shared_ptr`
Often, RAII classes can implement reference-counting copying behavior by containing a `shared_ptr` data member. For example, if `Lock` wanted to employ reference counting, it could change the type of `mutexPtr` from `Mutex*` to `shared_ptr<Mutex>`. Unfortunately, `shared_ptr`'s default behavior is to delete what it points to when the reference count goe to zero, and that's not what we want. When we're done with a `Mutex`, we want to unlock it, not delete it.
Fortunately, `shared_ptr` allows specification of a "delete" — a function or function objet to be called when reference count goes to zero. (This functionality does not exist for `auto_ptr`, which _always_ delete its pointer.) The deleter is an optional second parameter to the `shared_ptr` constructor, so the code would look like this:
```C++
class Lock {
public:
    // init shared_ptr with the Mutex to point to and the unlock function as the deleter
    explicit Lock(Mutex *pm)
    {
        lock(pm);
        mutexPtr.reset(pm, unlock);
    }
private:
    shared_ptr<Mutex> mutexPtr;    // use shared_ptr instead of raw pointer
```
**Note:** The `Lock` class non longer declares a destructor. We know that class's destructor (regardless of whether it is compiler-generated or user-defined) automatically invokes the destructors of the class's non-static data members. In this example, that's `mutexPtr`. But `mutexPtr`'s destructor will automatically call the `shared_ptr`'s deleter — `unlock`, in this case — when the mutex's reference count goes to zero.

* **Copy the underlying resource.** Sometimes you can have as many copies to a resource as you like, and the only reason you need a resource-managing class is to make sure that each copy is released when you're done with it. In that case, copying the resource-managing object should also copy the resource it wraps. That is, copying a resource-managing object performs a "deep copy".
**Example:** Some implementations of the standard string type consist of pointers to heap memory, where the characters making up the string are stored. Objects of such strings contain a pointer to the heap memory. When a string object is copied, a copy is made of both the pointer and the memory it points to. Such strings exhibit deep copying.

* **Transfer ownership of the underlying resource.** On rare occasion, you may wish to make sure that only one RAII object refers to a raw resource and that when the RAII object is copied, ownership of the resource is transferred from the copied object to the copying object.

**Things to Remember**
* Copying a RAII object entails copying the resource it manages, so the copying behavior of the resource determines the copying behavior of the RAII object.
* Common RAII class copying behavior are disallowing copying and performing reference counting, but other behaviors are possible.




