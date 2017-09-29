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






