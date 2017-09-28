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
Consider a `NamedObject` template that allows to associate names with objects of type `T`:
```C++
template<typename T>
class NamedObject {
public:
    NamedObject(const char* name, const T& value);
    NamedObject(const std::string& name, const T& value);
    ...
private:
    std::string nameValue;
    T objectValue;
};
```
Because a constructor is declared in `NamedObject`, compilers won't generate a default constructor. But `NamedObject` declares neither copy constructor nor copy assignment operator, so compilers will generate those functions (if needed). Look, then, at this use of copy constructor:
```C++
NamedObject<int> no1("Smallest prime Number",2);
NamedObject<int> no2(no1);                            // calls copy constructor
```
The compiler-generated copy assignment operator usually behave coorect when the resulting copde is both legal and has a reasonable chance of making sense. If either of these tests fails, compilers will refuse to generate an `operator=` for your class.
#####Example:
Suppose `NamedObject` were defined like this, where `nameValue` is a _reference_ to a string and `objectValue` is a _const_T:
```C++
template<typename T>
class NamedObject{
public:
    // this ctor no longer takes aconst name, because nameValue
    // is now a reference-to-non-const string. The char* constructor
    // is gone, because we must have a strng to refer to.
    NamedObject(std::string& name, const T& value);
    ...                        // as above, assume no operator= is decalared
private:
    std::string& nameValue;    // this is now a reference
    const T objectValue;       // this is now const
};
```
Now consider what should happen here:
```C++
std::string newDog("Persephone");
std::string oldDog("Satch");

NamedObject<int> p(newDog, 2);         // when I originally wrote this, our dog Persephone was about to have her second birthday
NamedObject<int> s(oldDog, 36);        // the family dog Satch (from my childhood) would be 36 if she were still alive
p=s;                                   // what should happed to the data members in p?
```
Before the assignment, both `p.nameValue` and `s.nameValue` refer to `string` objects, though not the same ones. How should assignment affect `p.nameValue`? After the assignment, should `p.nameValue` refer to the `string` referred to by `s.nameValue`, i.e., should the reference itself be modified? If so, that breaks new ground, because
> C++ doesn't provide a way to make a reference refer to a different object.
Alternatively, should the string object to which `p.nameValue` refers be modified, thus affecting other objects that hold pointers or references to that `string`, i.e., objects not directly involved in the assignment? Is that what the compiler-generated copy assignment operator should do?








