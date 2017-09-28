### Item 7 - Declare destructors virtual in polymorphic base classes.
There are alot of ways to keep track of time, so it would be reasonable to create a `TimeKeeper` base class along with derived classes for different apporaches to timekeeping:
```C++
class TimeKeeper {
public:
    TimeKeeper();
    ~TimeKeeper();
    ...
};

class AtomicClock: public TimeKeeper {...};
class WaterClock: public TimeKeeper {...};
class WristWatch: public TimeKeeper {...};
```
Many clients will want access to the time without worrying about the details of how it's calculated, 
So a **_factory function_**:
> A function that returns a base class pointer to a newly-created derived class object.

can be used to return a pointer to a timekeeping object:
```C++
TimeKeeper* getTimeKeeper();        // returns a pointer to a dynamically allocated
                                    // object of a class derived from TimeKeeper
```
In keeping with the conventions of the factory functions, the objects returned by `getTimeKeeper` are on the heap, so to avoid leaking memory and other resources, it's important that each returned object be properly deleted:
```C++
TimeKeeper* ptk = getTimeKeeper();  // get dynamically allocated object from TimeKeeper hierarchy
...                                 // use it
delete ptk                          // release it to avoid resource leak
```
The problem is that `getTimeKeeper` returns a pointer to a derived class object (e.g., `AtomicClock`), that object is being deleted via base class pointer (i.e., a `TimeKeeper*` pointer), and the base class (`TimeKeeper`) has a _non-virtual destructor_. This is a recipe for disaster, because C++ specifies that when a derived class ojbect is deleted through a pointer to a base class with a non-virtual destructor, results are undefined. What typically happens at runtime is that the derived part of the object is never destroyed. If a call to `getTimeKeeper` happened to return a pointer to an `AtomicClock` object, the `AtomicClock` part of the object (i.e., the data members declared in the `AtomicClock` class) would probably not be destroyed, nor would the `AtomicClock` destructor run. However, the base class part (i.e., the `TimeKeeper` part) typically would be destroyed, thus leading to a curious "partially destroyed" object. This is an excellent way to leak resources, corrupt data structures, and spend alot of time with a debugger.

To Eliminate this problem, give the base class a virtual destructor.
```C++
class TimeKeeper {
public:
    TimeKeeper();
    virtual ~TimeKeeper();
    ...
};
```
Base classes generally contain virtual functions other than the destructor, because the purpose of virtual functions is to allow customixation of derived class implementations. 
> Declare a virtual destructor in a class if and only if that class contains at least one virtual function.

If a class does not contain virtual functions, that often indicates it is not meant to be used as a base class. When a class is not intended to be a base class, making the destructor virtual is usually a bad idea. Consider a class for representing points in two-dimensional space:
```C++
class Point {            // a 2D point
public:
    Point(int xCoord, int yCoord);
    ~Point();
private:
    int x,y;
};
```
If an `int` occupies 32 bits, a `Point` object can typically fit into a 64-bit register. If `Point`'s destructor is made virtual, however, the situation changes.

The implementation of virtual functions requires that objects carry information that can be used at runtime to determine which virtual functions shoul dbe invoked on the object. This information typically takes thr form of a pointer called a `vptr` ("virtual table pointer"). The `vptr` points to an array of function pointers called a `vtbl` ("virtual table"); each class with virtual functions has an associated `vtbl`. When a virtual function is invoked on an object, the actual function called is determined by following the object's `vptr` to a `vtbl` and then looking up the approporiate function pointer in the `vtbl`.

If the `Point` class contains a virtual function, object of that type will increase in size. On a32-bit architecure, they'll go from 64 bits (for the 2 `int`s) to 96 bits (for the `int`s plus the `vptr`); on a 64-bit architecture, they may go from 64 to 128 bits, because pointers on such architectures are 64 bits in size.

It is possible to get bitten by the non-virtual destructor problem even in the complete absence of virtual functions.
#####For Example:
The standard `string` type contains no virtual functions, but misguided programmers sometimes use it as a base class anyway:
```C++
class SpecialString: public std::string {    // bad idea! std::string has a non-virtual destructor
};
```
Anywhere in an application you somehow convert a pointer-to-SpecialString into a pointer-to-string and you then use `delete` on the `string` pointer, you are instantly transported to the realm of undefined behavior:
```C++
SpecialString* pss = new SpecialString("Impending Doom");
std::string* ps;
...
ps = pss;                        // SpecialString* --> std::string*
...
delete ps;                       // undefined! In practice, *ps SpecialString resources will be 
                                 // leaked, becayse the SpecialString destructor won't be called.
```
The same analysis applies to any class lacking a virtual destructor, including all the STL container types (e.g., `vector`, `list`, `set`, `unordered_map`, etc.).









