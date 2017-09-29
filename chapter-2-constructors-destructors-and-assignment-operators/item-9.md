### Item 9 - Never call virtual functions during construction or destruction.
Suppose you've got a class hierarchy for modelling stock transactions, e.g., buy order, sell orders, etc. It's important that such transaction be auditable, so each time a transaction object is created, an appropriate entry needs to be created in an audit log. This seems like a reasonable way to approach the problem:
```C++
class Transaction {                             // base class for all transactions.
public:
    Transaction();
    virtual void logTransaction() const = 0;    // make type-dependent log entry
    ...
};
Transaction::Transaction()                    // implementation of base class ctor
{
    ...
    logTransaction();                        // as final action, log this transaction
}

class BuyTransaction: public Transaction    // derived class
{
public:
    virtual void logTransaction() const;    // how to log transactions of this type
    ...
};

class SellTransaction: public Transaction    // derived class
{
public:
    virtual void logTransaction() const;    // how to log transactions of this type
    ...
}
```
Consider what happens when this code is executed:
```C++
BuyTransaction b;
```
Clearly a `BuyTransaction` constructor will be called, but first, a `Transaction` constructor must be called; base class parts of derived class objects are constructed before derived class parts are. The last line of the `Transaction` constructor calls the virtual function `logTransaction`, but there is where the surprise comes in. The version of `logTransaction` that's called is the one in `Transaction`, not the one in `BuyTransaction` — even though the type of object being created is `BuyTransaction`. During base class construction, virtual functions never go down into derived classes.

In our example, while the `Transaction` constructor is running to initialize the base class part of a `BuyTransaction` object, the object is of type `Transaction`. That's how every parto of C++ will treat it, and the treatment makes sense: the `BuyTransaction` specific parts of the object haven't been initialized yet, so it's safest to treat them as if they didn't exist.
> An object doesn't become a derived class object until execution of a derived class constructor begins.

They same reasoning applies during destruction. Once a derived class destructor has run, the object's derived class data members assume undefined values, so C++ treats them as if they no longer exist. Upon entry to the base class destructor, the object becomes a base class object, and all parts of C++ — virtual functions, `dynamic_cast`, etc., — treat it that way.

It's not always so easy to detect calls to virtual functions during construction or destruction. if `Transaction` had multiple constructors, each of which had to perform some of the same work, it would be good software engineering to avoid code replication by putting the common initialization code, including the call to `logTransaction`, into a private non-virtual initialization function, say, `init`:
```C++
class Transaction {
public:
    Transaction()
    { init(); }                                // call to non-virtual
    virtual void logTransaction() const = 0;
    ...
private:
    void init()
    {
         ...
         logTransaction();                    // that calls a virtual!
     }
 };
```
This code is conceptually the same as the earlier version, but it's more insidious, because it will typically compile and link without complaint. In this case, because `logTransaction` is pure virtual in `Transacction`, most runtime systems will abort the program when the pure virtual is called. However, if `logTransaction` were a normal virtual function (i.e. not pure virtual) with an implementation in `Transaction`, that version would be called, and the program would merrily trot along, leaving you to figure out why the wrong version of `logTransaction` was called when a derived class object was created.

#### Reasonable Solution:
There are different ways to approach this problem. One is to turn `logTransaction` into a non-virtual function in `Transaction`, then require that derived class constructors pass the necessary log information to the `Transaction` constructor. That function can then safely call the non-virtual `logTransaction`. Like this:
```C++
class Transaction {
public:
    explicit Transaction( const std::string& logInfo);
    void logTransaction( const std::string& logInfo) const;    // now a non-virtual function
    ...
};
Transaction::Transaction( const std::string& logInfo)
{
    ...
    logTransaction(logInfo)                                    // now a non-virtual call
}
class BuyTransaction: public Transaction {
public:
    BuyTransaction(parameters): 
        Transaction( createLogString(parameters))             // passing log info to base class constructor
    { ... }
    ...
private:
    static std::string createLogString(parameters);
};
```
In other words, since you can't use virtual functions to call down from base classes during construction, you can compensate by having derived classes pass necessary construction information up to base class constructors instead.

In this example, note the use of the **_private static_** function **`createLogString`** in `BuyTransaction`. Using the helper function to create a value to pass to a base class constructor is often more convenient than going through contortions in the member initialization list ot give the base class what it needs.

**Things to Remember:**
* Don't call virtual fucntions during construction or destruction, because such calls will never go to a more derived class than that of the currently executing constructor or destructor.





