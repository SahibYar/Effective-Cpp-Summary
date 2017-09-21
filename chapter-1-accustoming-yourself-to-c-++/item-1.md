### Item 1 - View C++ as a federation of languages.
In the beginning, C++ was just  C with some object-oriented features tacked on, but today C++ is a _multiparadigm programming language_, one supoorting a combination of procedural, object-oriented, functional, generic and metaprogramming features.
The easiest way is to view C++ not as a single language but as a federation of related languages within a particular sublanguage, the rules tend to be simple, straightforward, and easy to remember. To make sense of C++, one has to recognize its primary sub-languages. Fortunately, there are only four:
* **C.** Way down step, C++ still based on C. Blocks, statements, the preprocessor, built-in data types, array, pointers, etc., all come from C. 
* **Object-Oriented C++.** This part of C++ is what C with Classes was all about: classes (including constructors and destructors), encapsulation, inheritance, polymorphism, virtual functions (dynamic binding), etc.This is the part of C++ to which the classic rules for object-oriented design most directly apply.
* **Template C++.** This is the generic programming part of C++, the one that most programmers have the least experience with. Template considerations pervade C++, and it's not uncommon for rules of good programming to include special template-only clauses. Infact, templates are so powerful, they give rise to a complete new programming paradigm, <b>Template Meta Programming <i>TMP</i></b>.
* **The STL.** The STL is a template library, of course, but it's a very special template library. Its conventions regarding containers, iterators, algorithms, and function objects mesh beautifullu, but templates and libraries can be built around other ideas too.

**Things to Remember:**
* Rules for effective C++ programming vary, depending on the part of C++ you are using.
