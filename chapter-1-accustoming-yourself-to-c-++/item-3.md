### Item 3 - Use `const` whenever possible
The `const` keyword is remarkably versatile. Outside of classes, it is use for constants at global or namesapces scope, as well as for objects declared `static` at file, function or block scope. Inside classes, you can use it for both static and non-static data members. For pointers, you can specify whether the pointer itself is `const`, the data it points to is `const`, both or neither:
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

When what's pointed to is constant, some programmers list `const` before the type. Others list it after the type. Others list it after the type but before the asterisk. There is no difference in meanining, so the following functions take the same parameter type:
```C++
void f1(const Widget *pw);      //  f1 takes apointer to constant Widget object
void f2(Widget const *pw);      //  so does f2
```
* In **STL** `iterator` acts much like a `T*` pointer. Declaring an `iterator` `const` is like declaring a pointer `const` (i.e. declaring a `T *const` pointer). The STL analogue to a `const T*` pointer is <b>`const_pointer`</b>

```C++
std::vector<int> vec;

const std::vector<int>::iterator iter = vec.begin();  //  iter acts like a T* const
*iter = 10;                                           //  OK, changes what iter points to. 
++iter;                                               //  error, iter is const

std::vector<int>::const_iterator cIter = vec.begin(); //  cIter acts like a const T*
*cIter = 10;                                          //  error! *cIter is const
++cIter;                                              //  fine, changes cIter
```
One of the most powerful uses of ```const``` is its application to <i>function declarations</i>. Within a function declaration,
* ```const``` can refer to the function return type,
* to individual parameters,
* and, for member functions, to the function as a whole.

#### `const` Member Functions
The purpose of `const` on member functions is to identify which member functions may be invoked on `const` objects. One of the fundamental ways to improve a C++ program's performance is to pass objects by **reference-to-const**. That technique is viable only if there are `const` member functions with which to manipulate the resulting const-qualified objects.
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
**Note:** The return *type* of the non-const `operator[]` is a *reference* to a `char` — a `char` itself would not do. if `operator[]` did return a simple `char`, statements like this wouldn't compile:
```C++
tab[0] = 'x';
```
That's because it's never legal to modify the return value of a function that returns a built-in type. Even if it were legal, the fact the C++ returns objects by value would mean the a *copy* of `tb.text[0]` would be modified, not `tb.text[0]` itself, and that's not the behaviour we want.
`TextBlock` `operator[]` can be used like this:
```C++
TextBlock tb("Hello");
std::cout << tb[0];       //  call non-const TextBlock::operator[]

void print(const TextBlock& ctb)  //  in this function, ctb is const
{
  st::cout << ctb[0];    //  call const TextBlock::operator[]
}
```
By overloading `operator[]` and giving the different versions different return _types_, you can have `const` and `non-const` `TextBlock` handled differently.
```C++
std::cout << tb[0];    //  fine - reading a non-const TextBlock
tb[0] = 'x';           //  fine - writing a non-const TextBlock
std::cout << ctb[0];   //  fine - reading a const TextBlock
ctb[0] = 'x';          //  error - writing a const TextBlock
```
**Note:** The error here has only to do with the return _type_ of the `operator[]` that is called; the calls to `operator[]` themselves are all fine. The error arises out of an attempt to make an assignment to a `const char&`, because that's the return type from the `const` version of `operator[]`.
#### Avoiding Duplication in `const` and non-`const` Member Functions
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
      const_cast<char&>(  // cast away const on operator[] return type;
        static_cast<const TextBlock&>(*this)  // add const to *this’s type;
          [position]  // call const version of op[]
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
