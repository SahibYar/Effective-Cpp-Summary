### Item 11 - Handle assignemtn to self in `operator=`.
An assignment ot self occurs when an object is assigned to itself:
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






