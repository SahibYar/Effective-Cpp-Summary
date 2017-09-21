# [Effective C++](https://www.amazon.com/Effective-Specific-Improve-Programs-Designs/dp/0321334876) by [Scott Meyers](https://en.wikipedia.org/wiki/Scott_Meyers) \(Short Summary\) {#effective-c-by-scott-meyers-short-summary}

---

Effective C++ is one of the best book for improving C++ programs and designs by using C++ best practices. And this book is recommended to every one pursuing career as a C++ developer, So I am writing down this short summary for my own personal use to have a short and to the point summary of the book for future use.

## Terminologies used in this book. {#terminologies-used-in-this-book}

---

#### Declaration {#declaration}

A _Declaration_ tells compiler about the name and type of something, but it omits certain details. For Example:

```cpp
extern int size;                         // object declaration
std::size_t numDigits(int number);       // function declaration
class Widget;                            // class declaration
template<typename T> class GraphNode;    // template delaration
```

#### Signature {#signature}

Each function's declaration reveals its _signature_ i.e. its parameter and return types. A function's signature is the same as its types. In the case of `numDigits`, the signature is `size_t(int)`, i.e. "Function taking an `int` and returning `std::size_t`."

#### Definition {#definition}

A _definition_ provides compilers with details a declaration omits. For an object, the definition is where compilers set aside memory for the object. For a function or a function template, the definition provides the code body. For a class or a class template, the definition lists the members of the class or template:

```cpp
int x                                   //  object definition

std::size_t numDigits(int number)       //  function definition
{
  std::size_t digitsSoFar = 1;
  while ((number /= 10) != 0)
    ++digitsSoFar;
  return digitsSoFar
}

class Widget                            //  class definition
{
public:
  Widget();
  ~Widget();
  .....
};

template<typename T>                    //  template definition
class GraphNode
{
public:
  GraphNode();
  ~GraphNode();
  .....
};
```

#### Initialization {#initialization}

_Initialization_ is the process of giving an object its first value. For objects generated from structs and classes, initialization is performed by constructors.

#### Default Constructors {#default-constructor}

A _Default Constructor_ is one the can be called without any arguments. Such a constructor has no parameters or has a default value for every parameter:

```cpp
class A
{
public:
  A();                                      // default constructor
};

class B
{
public:
  explicit B(int x=0, bool b=true);         //  default constructor
};

class C
{
public:
  explicit C(int x);                        //  not a default constructor
};
```

The constructor for `class B` and `class C` are declared <b>`explicit`</b> here. That prevent them from being used to perform implicit type conversions, though they may still be used for explicit type conversions, <b>`explicit`</b> is usually preferred because it prevent compilers from performing unexpected (often unintended) type conversions.

#### Copy Constructor vs Copy Assignment Operator {#copy-constructor-vs-copy-assignment-operator}
The _copy constructor_ is used to initialize an object with a different object of the same type, and the _copy assignment operator_ is used to copy the value from one object to another of the same type:
```C++
class Widget
{
public:
  Widget();                                 //  default constructor
  Widget(const Widget& rhs);                //  copy constructor
  Widget& operator=(const Widget& rhs);     //  copy assignment operator
  .....
};

Widget w1;                                  //  invoke default constructor
Widget w2(w1);                              //  invoke copy constructor
w1 = w2;                                    //  invoke copy assignment operator
Widget w3 = w2;                             //  invoke copy constructor
```
Fortunately, copy constructor is easy to distinguish from copy assignment. If a new object is being defined, a constructor has to be called; it can't be assignment. If no new object is being defined, no constructor can be involved, so it's an assignment.

### Standard Template Library (STL)
<i>STL</i> is the part of C++'s standard library devoted to [containers](http://en.cppreference.com/w/cpp/container) (e.g., [vector](http://en.cppreference.com/w/cpp/container/vector), [list](http://en.cppreference.com/w/cpp/container/list), [set](http://en.cppreference.com/w/cpp/container/set), [map](http://en.cppreference.com/w/cpp/container/map), etc), [iterators](http://en.cppreference.com/w/cpp/iterator) (e.g., ```vector<int>::iterator```, ```set<string>::iterator```, etc.), [algorithms](http://en.cppreference.com/w/cpp/algorithm) (e.g. ```for_each```, ```find```,```sort``` etc.), and related functionality.




