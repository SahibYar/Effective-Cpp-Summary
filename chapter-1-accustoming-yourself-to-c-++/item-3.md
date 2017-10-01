### Item 3 - Use `const` whenever possible
The `const` keyword is remarkably versatile. Outside of classes, it is use for constants at global or namespaces scope, as well as for objects declared `static` at file, function or block scope. Inside classes, you can use it for both static and non-static data members. For pointers, you can specify whether the pointer itself is `const`, the data it points to is `const`, both or neither:
```C++
char greeting[]="Hello";
char *p = greeting;             //  non-const pointer, non-const data
const char *p = greeting;       //  non-const pointer, const data
char *const p = greeting;       //  const pointer, non-const data
const char *const p = greeting; //  const pointer, const data
```
If the word `const` appears to the left of the asterisk, what's pointed to is constant.
If the word `const` appears to the right of the asterisk, the pointer _itself_ is constant.
if `const` appears on both sides, both are constant.

When what's pointed to is constant, some programmers list `const` before the type. Others list it after the type. Others list it after the type but before the asterisk. There is no difference in meaning, so the following functions take the same parameter type:
```C++
void f1(const Widget *pw);      //  f1 takes a pointer to constant Widget object
void f2(Widget const *pw);      //  so does f2
```
* In **STL** `iterator` acts much like a `T*` pointer. Declaring an `iterator` `const` is like declaring a pointer `const` (i.e. declaring a `T *const` pointer): the iterator isn't allowed to point to something different, but the thing it points to may be modified. If you want an iterator that points to something that can't be modified, the STL analogue to a `const T*` pointer is **`const_pointer`**

```C++
std::vector<int> vec;

const std::vector<int>::iterator iter = vec.begin();  //  iter acts like a T* const
*iter = 10;                                           //  OK, changes what iter points to. 
++iter;                                               //  error, iter is const

std::vector<int>::const_iterator cIter = vec.begin(); //  cIter acts like a const T*
*cIter = 10;                                          //  error! *cIter is const
++cIter;                                              //  fine, changes cIter
```
One of the most powerful uses of `const` is its application to _function declarations_. Within a function declaration,
* `const` can refer to the function return type,
* to individual parameters,
* and, for member functions, to the function as a whole.

---

Having a a function return a constant value is generally inappropriate, but sometimes doing so can reduce the incidence of client errors with out giving up safety or efficiency.

**Example:**

Consider the declaration of the operator* function for rational numbers:
```C++
class Ration { ... };
const Rational operator* (const Rational& lhs, const Rational& rhs);
```
Here the result of `operator*` be a `const` object, if it weren't, clients would be able to commit atrocities like this:
```C++
Rational a,b,c;
...
(a * b) = c;          // invoke operator= on the result of a*b!
```
It is a simple typo (and a type that can be implicitly converted to `bool`):
```C++
if (a * b = c) ...    // oops, meant to do a comparison!
```
Such code would be flat-out illegal if `a` and `b` were of a built-in type. Declaring `operator*` return value `const` prevents it, and that's why it's The Right Thing To Do in this case.

#### `const` Member Functions
The purpose of `const` on member functions is to identify which member functions may be invoked on `const` objects. Such member functions are important for two reasons:
1. They make the interface of a class easier to understand. It's important to know which functions may modify an object and which may not.
2. They make it possible to work with `const` objects.

One of the fundamental ways to improve a C++ program's performance is to pass objects by **reference-to-const**. That technique is viable only if there are `const` member functions with which to manipulate the resulting const-qualified objects.
Many people overlook the fact that member functions differing _only_ in their constness can be overloaded, but this is an important feature of C++. Consider a class for representing a block of text:
```C++
class TextBlock
{
public:
  ...
  const char& operator[](std::size_t position) const  //  operator[] for const objects
  {
    return text[position];
  }
  char& operator[](std::size_t position)             //  operator[] for non-const objects
  {
    return text[position];
  }
private:
  std::string text;
};
```
`TextBlock` `operator[]` can be used like this:
```C++
TextBlock tb("Hello");
std::cout << tb[0];       //  call non-const TextBlock::operator[]

const TextBlock ctb("World");
std::cout << ctb[0];      // call const TextBlock::operator[]
```
Incidentally, `const` objects most often arise in real programs as a result of being passed by _reference_ or _reference-to-const_. The example of `ctb` above is artificial. This is more realistic:
```C++
void print(const TextBlock& ctb)  //  in this function, ctb is const
{
  st::cout << ctb[0];    //  call const TextBlock::operator[]
}
```
By overloading `operator[]` and giving the different versions different return types, you can have `const` and `non-const` `TextBlock` handled differently.
```C++
std::cout << tb[0];    //  fine - reading a non-const TextBlock
tb[0] = 'x';           //  fine - writing a non-const TextBlock
std::cout << ctb[0];   //  fine - reading a const TextBlock
ctb[0] = 'x';          //  error - writing a const TextBlock
```
**Note:** The error here has only to do with the return type of the `operator[]` that is called; the calls to `operator[]` themselves are all fine. The error arises out of an attempt to make an assignment to a `const char&`, because that's the return type from the `const` version of `operator[]`.


**Note:** The return *type* of the non-const `operator[]` is a *reference* to a `char` — a `char` itself would not do. if `operator[]` did return a simple `char`, statements like this wouldn't compile:
```C++
tab[0] = 'x';
```
That's because it's never legal to modify the return value of a function that returns a built-in type. Even if it were legal, the fact the C++ returns objects by value would mean the a *copy* of `tb.text[0]` would be modified, not `tb.text[0]` itself, and that's not the behavior we want.
---
**What does it mean for member function to be a `const`?** There two prevailing notions: _bitwise constness_ (also know as _physical constness_) and _logical constness_.

The bitwise `const` camp believes that a member function is `const` if and only if it doesn’t modify any of the object’s data members (excluding those that are static), i.e., if it doesn’t modify any of the bits inside the object. The nice thing about bitwise constness is that it’s easy to detect violations: compilers just look for assignments to data members. In fact, bitwise constness is C++’s definition of constness, and a const member function isn’t allowed to modify any of the non-static data members of the object on which it is invoked.
Unfortunately, many member functions, that don't act very `const` pass the bitwise test. In particular, a member function that modifies what a pointer _points_ to frequently doesn't act `const`. But if only the _pointer_ is in the object, the function is bitwise `const`, and compiler won't complain.

**Example:**

Suppose we have a `TextBlock` like class that stores its data as a `char*` instead of a `string`, because it needs to communicate through a C API that doesn't understand `string` objects.
```C++
class CTextBlock {
public:
  ...
  // inappropriate (but bitwise const) declaration of operator[]
  char& operator[] (std::size_t position) const
  { return pText[position]; }
private:
  char* pText;
};
```
This class (inappropriately) declares `operator[]` as a `const` member function, even though that function returns a reference of the object's internal data. And note that `operator[]`s implementation doesn't modify `pText` in any way. As a result, compilers will happily generate code for `operator[]`; it is, after all, bitwise `const`, and that's all compilers check for. But look what it allowd to happen:
```C++
const CTextBlock cctb("Hello");      // declare constant object
char* pc = &cctb[0];                 // call the const operator[] to get a pointer to cctb's data
*pc = 'J';                           // cctb now has the value "Jello"
```
Surely there is something wrong when you create a constant object with a particular value and you invoke only `const` member functions on it, yet you still change its value!
This leads to notion of logical constness. Adherents to this philosophy — and you should be among them — argue that a `const` member function might modify some of the bits in the object on which it's invoked, but only in ways that clients cannot detect.

**Example:**
Your `CTextBlock` class might want to cache the length of the textblock whenever it's requested:
```C++
class CTextBlock {
public:
  ...
  std::size_t length() const;
private:
  char* pText;
  std::size_t textLength;        // last calculated length of textblock
  bool lengthIsValid;            // whether length is currently valid
};

std::size_t CTextBlock::length() const
{
  if (!lengthIsValid) {
    // error! can't assign to textLength and lengthisValid in a const member function.
    textLength = std::strlen(pText);
    lengthIsValid = true;
  }
  return textLength;
}
```
This implementation of `length` is certainly not bitwise `const` — both `textLength` and `lengthIsValid` may be modified — yet it seems as though it should be valid for `const CTextBlock` objects. Compilers disagree. They insist on bitwise constness. What to do?

The solution is simple: take advantage of C++'s `const`-related wiggle room known as **`mutable`**. `mutable` frees non-static data members from the constraints of bitwise constness.
```C++
class CTextBlock {
public:
  ...
  std::size_t length() const;
private:
  char* pText;
  // these data members may always be modified, even in const member functions.
  mutable std::size_t textLength;
  mutable bool lengthIsValid;
};

std::size_t CTextBlock::length() const
{
  if (!lengthIsValid) {
    textLength = std::strlen(pText);   // now fine
    lengthIsValid = true;              // also fine
  }
  return textLength;
}
```

#### Avoiding Duplication in `const` and Non-`const` Member Functions
```C++
class TextBlock 
{
public:
  ...
  const char& operator[](std::size_t position) const
  {
    ...      // do bounds checking
    ...      // log access data
    ...      // verify data integrity
    return text[position];
  }

  char& operator[](std::size_t position)
  {
    ...      // do bounds checking
    ...      // log access data
    ...      // verify data integrity
    return text[position];
  }
private:
 std::string text;
};
```
To avoid code duplication, it is possible to move all the code for bound checking, etc. into a separate member function that both versions of `operator[]` call, but you have still got the duplicated calls to that function and you've still got the duplicated `return` statement code. The `const` version of `operator` does exactly what the non-`const` version does, it just a `const`- qualified `return` type.

Casting away the `const` on the `return` value is safe, in this case because whoever called the non-`const` `operator[]` must have had a non-`const` object in the first place. Otherwise they couldn't have called a non-`const` function. So having the non-`const` `operator[]` call the `const` version is a safe way to avoid code duplication, even though it requires a cast.
```C++
class TextBlock 
{
public:
  ...
  const char& operator[](std::size_t position) const // same as before
  {
    ...
    ...
    ...
    return text[position];
  }
  
  char& operator[](std::size_t position) // now just calls const op[]
  {
    return 
      const_cast<char&>(                      // cast away const on operator[] return type;
        static_cast<const TextBlock&>(*this)  // add const to *this’s type;
          [position]                          // call const version of op[]
      );
  }
  ...
};
```
This code has two casts, not one. We want the non-`const` `operator[]` to call the `const` one, but if, inside the non-`const` `operator[]`, we just call `operator[]`, we will recursively call ourselves. To avoid infinite recursion, we have to specify that we want to call the `const` `operator[]`, but there's no direct way to do that. Instead, we cast `*this` (so that our call to `operator[]` will call the `const` version) which is just forcing a safe conversion (from a non-`const` object to a `const` one), so we use a **`static_cast`**, the second to remove the `const` from the `const` `operator[]` `return` value via **`const_cast`**.

**Things to Remember:**
* Declaring something `const` helps compilers detect usage errors. `const` can be applied to objects at any scope, to function parameters and return types, and to member functions as a whole.
* Compilers enforce bitwise constness, but you should program using conceptual constness.
* When `const` and non-`const` member functions have essentially identical implementations, code duplication can be avoided by having the non-`const` version call the `const` version.
