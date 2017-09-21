### Item 2 - Prefer `const`, `enum`, and `inline` to `#defines`.

For simple constants, prefer `const` objects or `enums` to `#defines`.

```#define ASPECT_RATIO 1.653```

```const double AspectRatio = 1.653;```

When replacing `#defines` with constants, 2 cases are worth mentionining.

* The 1st one is constant pointers
```const char* const authorName="Scott Meyers";```
```const std::string authorName("Scott Meyers");```
`string` objects are generally preferable to their `char*` based progenitors.

* The 2nd special case concerns class-specific constants.



```C++
class CostEstimate
{
private:
  static const double FudgeFactor;                // declaration of static class constant; goes in header file
  ....
};

const double CostEstimate::FudgeFactor = 1.35;    // definition of static class constant; goes in implimentation file

```

This is all you need almost all the time, The only exception is when the value of a class constant is needed during compilation of the class, such as in the declaration of the array `GamePlayer::scrores` given below.
```C++
class GamePlayer
{
private:
  static const int NumTurns = 5;                  // constant declaration
  int scores[NumTurns];                           //  use of constant
  ...
};
```
Most old compiler do not accept above systax, Then accepted way to compensate for compilers that (incorrectly) forbid the in-class specification of initial values for static integral type (e.g., `int`, `char`, `bool`) class constants is to use what is affectionately known as **the enum hack**. This technique takes advantage of the fact that the values of an `enum` type can be used where `int` are expected. So `GamePlayer` could just as well be defined like this:
```C++
class GamePlayer
{
private:
  enum {NumTurns = 5};                            //  "the enum hack" - makes NumTurns a symbolic name of 5
  int scores[NumTurns];                           //  fine
  ...
};
  
```
Getting back to the _preprocessor_, another common (mis)use of the `#define` directive is using it to implement macros that look like functions but that don't incur the overhead of a function call. for example
```C++
// call function f with maximum of a and b
#define CALL_WITH_MAX(a,b) f((a)>(b)?(a):(b))
```
Macros like this have so many drawbacks, just thinking about them is painful
```C++
template <typename T>                                 //  because we don't know what T is, we
inline void callWithMax(const T& a, const T& b)       //  pass by reference-to-const
{
  f(a>b ? a:b);
}
```
**Things to Remember:**
* For simple constants, prefer ```const``` objects or ```enum``` to ```#define```.
* For function-like macros, prefer ```inline``` functions to ```#define```.
