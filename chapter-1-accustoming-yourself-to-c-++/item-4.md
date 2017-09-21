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








