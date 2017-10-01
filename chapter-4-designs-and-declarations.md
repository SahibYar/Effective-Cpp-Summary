## Chapter 4: Designs and Declarations

Software designs — approaches to getting the software to do what you
want it to do — typically begin as fairly general ideas, but they eventually become detailed enough to allow for the development of specific interfaces. These interfaces must then be translated into C++ declarations. In this chapter, we attack the problem of designing and declaring good C++ interfaces. We begin with perhaps the most important guideline about designing interfaces of any kind: that they should be easy to use correctly and hard to use incorrectly. That sets the stage
for a number of more specific guidelines addressing a wide range of topics, including correctness, efficiency, encapsulation, maintainability, extensibility, and conformance to convention.

The material that follows isn’t everything you need to know about
good interface design, but it highlights some of the most important considerations, warns about some of the most frequent errors, and provides solutions to problems often encountered by class, function, and template designers.

[**Item 18:**](https://sahibyar.gitbooks.io/effective-cpp-summary/content/) **Make interfaces easy to use correctly and hard to use incorrectly.**

[**Item 19:**](https://sahibyar.gitbooks.io/effective-cpp-summary/content/) **Treat class design as type design.**

[**Item 20:**](https://sahibyar.gitbooks.io/effective-cpp-summary/content/) **Prefer pass-by-reference-to-const tp pass-by-value.**

[**Item 21:**](https://sahibyar.gitbooks.io/effective-cpp-summary/content/) **Don't try to return a refernce when you must return an object.**

[**Item 22:**](https://sahibyar.gitbooks.io/effective-cpp-summary/content/) **Declare data members private.**

[**Item 23:**](https://sahibyar.gitbooks.io/effective-cpp-summary/content/) **Prefer non-member non-friend functions to member functions.**

[**Item 24:**](https://sahibyar.gitbooks.io/effective-cpp-summary/content/) **Declare non-member functions when type conversions should apply to all parameters.**

[**Item 25:**](https://sahibyar.gitbooks.io/effective-cpp-summary/content/) **Consider support for a non-throwing swap.**