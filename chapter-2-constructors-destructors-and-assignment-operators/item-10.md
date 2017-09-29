### Item 10 - Have assignment operator return a reference to `*this`.
One of the interesting things about assignemnts is that you can chain them together:
```C++
int x,y,z;
x = y = z = 15;                    // chain of assignments.
```
Also interesting is that assignment is right-associative, so the above assignment chain is parsed like this:
```C++
x = (y = (z = 15));
```
The way this is implemented is that assignment returns a reference to its left-hand argument, and that's the convention you should follow when you implement assignment operators for you classes:
```C++
class Widget {
public:
    ...
    Widget& operator= (const Widget& rhs)        // return type is a reference to the current class
    {
        ...
        return *this;
    }
    ...
};
```
This convention applies to all assignment operators, not just the standard form shown above. Hence:
```C++
class Widget {
public:
    ...
    Widget& operator+=(const Widget& rhs)        // it applies even if the operator's parameter type is unconventional
    {
        ...
        return *this;
    }
    ...
};
```
This is only a conventionl code that doesn't follow it will compile. However, the convention is followed by all the built-in types as well as by all the types in the STL.

**Things to Remember**
* Have assignment operators return a reference to `*this`




