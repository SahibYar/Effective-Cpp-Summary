### Item 11 - Handle assignment to self in `operator=`.
An assignment to self occurs when an object is assigned to itself:
```C++
class Widget { ... };
Widget w;
...
w = w;                        // assignment to self
```
This looks silly, but it's legal, so rest assured that clients will do it. Besides, assignment isn't always so recognizable. For example,
```C++
a[i] = a[j]                    // potential assignment to self
```
is an assignment to self if `i` and `j` have the same value, and
```C++
*px = *py;                    // potential assignment to self
```
is an assignment to self if `px` and `py` happen to point to the same thing. These less obvious assignments to self are the result of _aliasing_
> Having more than one way to refer to an object.

In general, code that operates on references or pointers to multiple objects of the same type needs to consider that the objects might be the same. In fact, the two objects need not even be declared to be of the same type if they're from the same hierarchy, because a base class reference or pointer can refer or point to an object of a derived class type:
```C++
class Base { ... };
class Derived: public Base { ... };
void doSomething(const Base& rb, Derived* pd);        //rb and *pd might actually be the same object,
```
You can fall into the trap of accidentally releasing a resource before you're done using it. For example, suppose you create a class that holds a raw pointer to a dynamically allocated bitmap:
```C++
class Bitmap { ... };
class Widget {
    ...
private:
    Bitmap *pb;                // ptr to a heap-allocated object
};
```
Here's an implementation of `operator=` that looks reasonable on the surface but is unsafe in the presence of assignment to self. (It's also not **_exception-safe_**, but we'll deal with that in a moment.)
```C++
Widget& Widget::operator= (const Widget& rhs)    // unsafe implementation of operator=
{
    delete pb;                    // stop using current bitmap
    pb = new Bitmap(*rhs.pb);     // start using a copy of rhs's bitmap
    return *this;
}
```
The self-assignment problem here is that inside `operator=`, `*this` (the target of the assignment) and `rhs` could be the same object. When they are, the `delete` not only destroys the bitmap for current object, it destroys the bitmap for `rhs`, too. At the end of the function, the `Widget` — which should not have been changed by the assignment to self — finds itself holding a pointer to a deleted object!<sup>[1](#myfootnote1)</sup>

The traditional way to prevent this error is to check for assignment to self via an _identity test_ at the top of `operator=`:
```C++
Widget& Widget::operator= (const Widget& rhs)
{
    if (this == &rhs)         // identity test: if a self-assignment,
        return *this;         // do nothing.
        
    delete pb;
    pb = new Bitmap(*rhs.pb);
    return *this;
}
```
This works, because it is self-assignment safe, but it is exception-unsafe. In particular, if the `new Bitmap` expression yields an exception (either because there is insufficient memory for allocation or because Bitmap's copy constructor throws one), the `Widget` will end up holding a pointer to a deleted Bitmap.<sup>[1](#myfootnote1)</sup>Such pointers are toxic. You can't safely delete them. You can't even safely read them.

Happily, making `operator=` exception-safe typically renders it self-assignment-safe too. As a result, it's increasingly common to deal with issues of self-assignment by ignoring them, focusing instead on achieving exception safety. Here, for example, we just have to be careful not to delete `pb` until after we've copied what it points to:
```C++
Widget& Widget::operator= (const Widget& rhs)
{
    Bitmap *pOrig = pb;            // remember original pb
    pb = new Bitmap(*rhs.pb);      // point pb to a copy of rh's bitmap
    delete pOrig;                  // delete the original pb
    
    return *this;
}
```
Now, if "new Bitmap" throws an exception, `pb` (and the `Widget` it's inside of) remain unchanged. Even without the identity test, this code handles assignment to self, because we make a copy of the original bitmap.

**Things to Remember**
* Make sure `operator=` is well-behaved when an object is assigned to itself. Techniques include comparing addresses of source and target objects, careful statement ordering, and copy-and-swap.
* Make sure that any function operating on more than one object behaves correctly if two or more of the objects are the same.

<a name="myfootnote1">1</a>: Probably. C++ implementations are permitted to change the value of a deleted pointer (e.g., to null or some other special bit pattern).






