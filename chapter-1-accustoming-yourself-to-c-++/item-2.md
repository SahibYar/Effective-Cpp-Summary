### Item 2 - Prefer `const`, `enum`, and `inline` to `#define`.
> Prefer the compiler to the preprocessor.

`#define` may be treated as if it's not part of the language. That's one of its problems. When you do something like this,
```C++
#define ASPECT_RATIO 1.653
```
the symbolic name `ASPECT_RATIO` may never be seen by compilers; it may be removed by the preprocessor before the source code ever gets to a compiler. As a result, the name `ASPECT_RATIO` may not get entered int the symbol table. This can be confusing if you get an error during compilation involving the use of the constant, because the error message may refer to `1.653`, not `ASPECT_RATIO`. If `ASPECT_RATIO` were defined in a header file you didn't write, you'd have no idea where that `1.653` came from, and you'd waste time tracking it down.
The solution is to replace the macro with a constant:
```C++
// uppercase names are usually for macros, hence the name change
const double AspectRatio = 1.653;
```
As a language constant, `AspectRatio` is definitely seen by compilers and is certainly entered into their symbol tables.

When replacing `#defines` with constants, 2 cases are worth mentioning.

* The first one is constant pointers. Because constant definitions are typically put in header files (where many different source files will include them), it's important that the _pointer_ be declared `const`, usually in addition to what the pointer points to.
**Example:** To define a constant `char*` based string in a header file, you to write `const` twice:
```C++
const char* const authorName = "Scott Meyers";
```
It's worth reminding you here that `string` objects are generally preferable to their `char*` based progenitors, so `authorName` is often better defined this way:
```C++
const std::string authorName("Scott Meyers");
```

* The 2nd special case concerns class-specific constants. To limit the scope of a constant to a class, you must make it a member, and to ensure there's at most one copy of the constant, make it a _static_ member.
```C++
class GamePlayer {
private:
  static const int NumTurns = 5;    // constant declaration
  int scores [NumTurns];
  ...
};
```
What you see above is a _declaration_ for `NumTurns`, not a definition. Usually C++ requires that you provide a definition for anything you use, but class-specific constants that are static and of integral type(e.g., integers, chars, bools) are an exception. As long as you don't take their address, you can declare them and use them without providing a definition. If you do take the address of a class constant, or if you compiler incorrectly insists on a definition even if you don't take the address, you provide a separate definition like this:
```C++
const int GamePlayer::NumTurns;    // definition of NumTurns
```
You put this in an implementation file, not a header file. Because the initial value of a class constants is provided where the constant is declared (e.g., `NumTurns` is initialized to 5 when it is declared), no initial value is permitted at the point of definition.

Older compilers may not accept the syntax above, because it used to be illegal to provide an initial value for a static class member at this point of declaration. Furthermore, in-class initialization is allowed only for integral types and only for constants. In cases where above syntax can't be used, you put the initial value at the point of definition:
```C++
class CostEstimate
{
private:
  static const double FudgeFactor;                // declaration of static class constant; goes in header file
  ...
};

const double CostEstimate::FudgeFactor = 1.35;    // definition of static class constant; goes in implementation file
```
This is all you need almost all the time, The only exception is when the value of a class constant is needed during compilation of the class, such as in the declaration of the array `GamePlayer::scores` above (where compilers insist on knowing the size of the array during compilation). Then accepted way to compensate for compilers that (incorrectly) forbid the in-class specification of initial values for static integral type (e.g., `int`, `char`, `bool`) class constants is to use what is affectionately known as **the enum hack**. This technique takes advantage of the fact that the values of an `enum` type can be used where `int` are expected. So `GamePlayer` could just as well be defined like this:

```C++
class GamePlayer
{
private:
  enum {NumTurns = 5};                            //  "the enum hack" - makes NumTurns a symbolic name of 5
  int scores[NumTurns];                           //  fine
  ...
};
```
The `enum` hack behaves in some ways more like a `#define` than a `const` does, and sometimes that's what you want. For example, it's legal to take the address of a `const`, but it's not legal to take the address of an `enum`, and it's typically not legal to take the address of a `#define`, either. If you don't want to let people get a pointer or reference to one to your integral constants, an `enum` is a good way to enforce that constraint.

Getting back to the _preprocessor_, another common (mis)use of the `#define` directive is using it to implement macros that look like functions but that don't incur the overhead of a function call. for example

```C++
// call function f with maximum of a and b
#define CALL_WITH_MAX(a,b) f((a)>(b)?(a):(b))
```

Macros like this have so many drawbacks, just thinking about them is painful
Whenever you write this kind of macro, you have to remember to parenthesize all the arguments in the macro body. Otherwise you can run into trouble when somebody calls the macro with an exception. But even if you get that right, look at the weird things that can happen:
```C++
int a=5, b=0;
CALL_WITH_MAX (++a, b);          // a is incremented twice
CALL_WITH_MAX (++a, b+10);       // a is incremented once
``` 
Here, the number of times that `a` is incremented before calling `f` depends on what it is being compared with!

Fortunately, you don't need to put up with this nonsense. You can get all the efficiency of a macro plus all the predictable behavior and type safety of a regular function by using a template for an inline function
```C++
template <typename T>                                 //  because we don't know what T is, we
inline void callWithMax(const T& a, const T& b)       //  pass by reference-to-const
{
  f(a>b ? a:b);
}
```
This template generates a whole family of functions, each of which
takes two objects of the same type and calls `f` with the greater of the two objects. There’s no need to parenthesize parameters inside the function body, no need to worry about evaluating parameters multiple times, etc. Furthermore, because `callWithMax` is a real function, it obeys scope and access rules. For example, it makes perfect sense to talk about an inline function that is private to a class. In general, there’s just no way to do that with a macro.

**Things to Remember:**
* For simple constants, prefer `const` objects or `enum` to `#define`.
* For function-like macros, prefer `inline` functions to `#define`.



