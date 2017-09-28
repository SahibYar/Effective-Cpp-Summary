### Item 4: Make sure that objects are initialized before they're used.

Reading uninitialized values yields undefined behaviour.

```C++
int x;    // It may or may not be initialized to (zero)

class Point
{
  int x,y;
};
...
Point p;    //  p's data members are sometimes guarenteed to be initialized to (zero)
            //  but sometimes they're not.
```

Always initialize your objects before using them. For non-memeber objects of built-in types, do this manually. For example:

```C++
int x = 0;                              //  manual initialization of an int
const char* text = "A C-style string";  //  manunal initialization of a pointer

double d;                               //  "initialization" by reading from an input stream
std::cin >> d;
```

The resposibility for initialization falls on **constructors**. Make sure that all constructors initialize everything in the object.

**Note:** Do not confuse _assignment_ with _initialization_.

```C++
class PhoneNumber { ... };
class ABEntry                           //  ABEntry = "Address Book Entry"
{
public:
  ABEntry (const std::string& name, const std::string& address, const std::list<PhoneNumber>& phones);
 private:
  std::string theName;
  std::string theAddress;
  std::list<PhoneNumber> thePhones;
  int numTimesConsulted;
};
ABEntry::ABEntry(const std::string& name, const std::string& address, const std::list<PhoneNumber>& phones)
{
  theName = name;                       //  these are all assignments not initializations
  theAddress = address;
  thePhones = phones;
  numTimesConsulted = 0;
}
```

This will yield `ABEntry` objects with the values you expect, but it's still not the best approach. The rules of C++ stipulate the data members of an object are initialized _before_ the body of a constructor is entered. Inside the `ABEntry` constructor, `theName`, `theAddress`, and `thePhones` aren’t being **initialized**, they’re being **assigned**. Initialization took place earlier — when their default constructors were automatically called prior to entering the body of the `ABEntry` constructor. This isn’t true for `numTimesConsulted`, because it’s a built-in type. For it, there’s no guarantee it was initialized at all prior to its assignment.  
A better way to write the `ABEntry` constructor is to use the member initilization list instead of assignments:

```C++
ABEntry::ABEntry(const std::string& name, const std::string& address, const std::list<PhoneNumber>& phones): 
        theName(name),
        theAddress(address),            // these are now all initializations
        thePhones(phones),
        numTimesConsulted(0)
      {}                                // the ctor body is now empty
```

This constructor yields the same end results as the one above, but it will often be more efficient. The assignment-based version first called default constructors to initialize the `theName`, `the Address`, and `thePhones`, then promptly assigned new values on top of the default-constructed ones. All the work performed in those default constructions was therefore wasted. The member initialization list apporach avoids that problem, because the arguments in the initialization list are used as constructors arguments for the various data members. In this case `theName` is copy-constructed from `name`, `theAddress` is copy-constructed from `phones`. For most types, a single call to copy constructor is more efficient - sometimes _much_ more efficient - than  a call to the default contructor followed by a call to the copy assignment operator.

If `ABEntry` had a constructor taking no parameters, it could be implemented like this:

```C++
ABEntry::ABEntry( ) : theName(),                // call theName’s default ctor;
                      theAddress(),             // do the same for theAddress;
                      thePhones(),              // and for thePhones;
                      numTimesConsulted(0)      // but explicitly initialize
                    {}                          // numTimesConsulted to zero
```

For objects of built-in type like `numTimesConsulted`, there is no difference in cost between initialization and assignment, but for consistency, it's often best to initialize everything via member initialization.

One aspect of C++ that isn't fickle is the order in which an object's data is initialized.

> Base classes are initialized before derived classes \([Item 12](https://sahibyar.gitbooks.io/effective-cpp-summary/content/chapter-2-constructors-destructors-and-assignment-operators/item-12.html)\), and within class, data members are initialized in the order in which they are declared.

In `ABEntry`, for example, `theName` will always be initialized first, `theAddress` second, `thePhones` third and `numTimesConsulted` last. This is true even if they are listed in a different order on the member initialization list.

Now, there is one more thing to be remembered, and that is,

#### The order of initialization of non-local static objects defined in different translation units. {#the-order-of-initialization-of-non-local-static-objects-defined-in-different-translation-units}

Let's pick that phrase apart it by bit.

A _**static object**_ is one that exists from the time it's constructed until the end of the program. Stack and heap-based objects are thus excluded. Included are global objects, objects defined at namespace scope, objects declared static inside classes, objects declared static inside functions, and object declared static at file scope. Static objects inside functions are knows as _local static objects_ \(because they're local to a function\), and the other kinds of static objects are known as _non-local static objects._ Static objects are destroyed when program exits, i.e. their destructors are called when main finishes executing.

A _**translation unit**_ is the source code giving rise to a single object file.It's basically a single source file, plus all of its `#include` files.

The problem we're concerned with, then, involves at least 2 separately compiled source files, each of which contains at least one non-local static object. And the actual problem is this: if initialization of a non-local static object in one translation unit uses a non-local static object in a different translation unit, the object it uses could be uninitialized, because _**the relative order of initialization of non-local static object defined in different translation units is undefined.**_

##### Example:

Suppose you have a `FileSystem` class that makes files on the Internet look like they're local. Since your class makes the world look like a single file system, you might create a special object at global or namespace scope representing the single file system:

```C++
class FileSystem {                // from your library's header file
public:
  ...
  std::size_t numDisk() const:    // one of many member functions
  ...
};

extern FileSystem tfs;            // declare object for clients to use("tfs"="the file system");
                                  // definition is in some.cpp in your library
```

Now suppose some client creates a class for directories in a file system. Naturally, their class uses the `tfs` object:

```C++
class Directory {                      // created by library client
public:
  Directory (params);
  ...
};

Directory::Directory(params)
{
  ...
  std::size_t disks = tfs.numDisks();  // use the tfs object
  ...
}
```

Further suppose this client decides to create a single `Directory` object for temporary files:

```C++
Directory tempDir(params);          // directory for temporary files
```

Now the imporatance of initialization order becomes apparent: unless `tfs` is initialized before `tempDir`, `tempDir`'s constructor will attempt to use `tfs` before it's been initialized. But `tfs` and `tempDir` were created by different people at different times in different source files — they're non-local static objects defined in different translation units. How can you be sure that `tfs` will be initialized before `tempDir` ?

Fortunately, a small design change eliminates the problem entirely. All that has to be done is to move each non-local static object into its own function, where it's declared **`static`** .These functions return references to the objects they contain. Clients then call the functions instead of referring to the objects. In other words, non-local static objects are replaced with _local_ static objects.

This approach is founded on C++'s guarantee that local static objects are initialized when the object's definition is first encountered during a call to that function. So if you replace direct accesses to non-local static objects with calls to functions that return references to local static objects, you're guarenteed that the references you get back will refer to initialized objects.

Here's the technique applied to both `tfs` and `tempDir`:

```C++
class FileSystem {...};                        //  as before
FileSystem& tfs()                              //  this replaces the tfs object; it could be static in the FileSystem class
{
  static FileSystem fs;                        //  define and initize a local static object
  return fs;                                   //  return reference to it.
}

class Directory {...};                        //  as before
Directory::Directory(params)                  //  as before, except references to tfs are now to tfs()
{
  ...
  std::size_t disks = tfs().numDisks();
  ...
}
Directory& tempDir()                          //  this replaces the tempDir object; it could be static in the Directory class
{
  static Directory td(param);                 //  define/initialize local static object return reference to it.
  return td;                                  //  return reference to it.
}
```


