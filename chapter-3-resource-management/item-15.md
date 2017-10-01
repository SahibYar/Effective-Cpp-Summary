### Item 15 - Provide access to raw resources in resource managing classes.
[Item 13](https://sahibyar.gitbooks.io/effective-cpp-summary/content/chapter-3-resource-management/item-13.html) introduces the idea of using smart pointers like `auto_ptr` or `shared_ptr` to hold the result of a call to a factory function like `createInvestment`:
```C++
std::shared_ptr<Investment> pInv (createInvestment());
```
Suppose that a function you'd like to use when working with `Investment` objects is this:
```C++
int daysHeld(const Investment* pi);    // return number of days investment has been held
```
You'd like to call it like this
```C++
int days = daysHeld(pInv);    // error!
```
but the code won't compile: `daysHeld` wants a raw `Investment*` pointer, but you're passing an object of type `shared_ptr<Investment>`.
You need a way to convert an object of the RAII class (in this case `shared_ptr`) into the raw resource it contains (e.g., the underlying `Investment*`). There are two general ways to do it: _explicit_ conversion and _implicit_ conversion.
`shared_ptr` and `auto_ptr` both offer a get member function to perform an _explicit_ conversion, i.e., to return (a copy of) the raw pointer inside the smart pointer object:
```C++
int days = daysHeld(pInv.get());        // fine, passes the raw pointer in pInv to daysHeld
```
Like virtually all smart pointer classes `shared_ptr` and `auto_ptr` also overload the pointer dereferencing operators (`->` and `*`), and this allows implicit conversion to the underlying raw pointers:
```C++
class Investment {            // root class for a hierarchy of investment types
public:
    bool isTaxFree() const;
    ...
};
Investment* createInvestment();    // factory function
shared_ptr<Investment> pi1(createInvestment());    // shared_ptr manage a resource
bool taxable1 = !(pi1->isTaxFree());        // access resource via operator->
...
auto_ptr<Investment> pi2(createInvestment()):    // auto_ptr manage a resource
bool taxable2 = !((*pi2).isTaxFree());     // access resource via operator*
```









.