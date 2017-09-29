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
```