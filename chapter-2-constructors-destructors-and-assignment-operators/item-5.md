### Know what functions C++ silently writes and calls
Empty class is not an empty class. If you don't declare them yourself, compilers will declare their own versions of a copy contructor, a copy assignment operator, and a destructor. Furthermore, if you declare no constructors at all, compilers will also declare a default constructor for you. All these functions will be both public and inline. As a result if you write
```C++
class Empty{};
```
it's essentially the same as if you'd written this:
```C++
class Empty {
public:
    Empty(){...}                                // default constructor
    Empty(const Empty& rhs){...}                // copy constructor
    ~Empty(){...}                               // destructor — see below for whether it's virtual
    Empty& operator=(const Empty& rhs){...}     // copy assignment operator
};
```
These functions are generated only if they are needed. The following code will cause each function to be generated:
```C++
Empty e1;                    // default constructor;
                             // destructor
Empty e2(e1);                // copy constructor
e2=e1;                       // copy assignment operator
```
**Note:** The generated destructor is non-virtual unless it's for a class inheriting from a base class that itself declares a virtual destructor (in which case the function's virtualness comes from the base class).

As for the copy constructor and copy assignment operator, the compiler -generated versions simply copy each non-static data member of the source object over to the target object.
#####Example: