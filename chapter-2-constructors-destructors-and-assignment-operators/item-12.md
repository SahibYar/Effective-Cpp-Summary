### Item 12 - Copy all parts of an object
Consider a class representing customers, where the copying functions have been manually written so that calls to them are logged:
```C++
void logCall(const std::string& funcName);    //make a log entry
class Customer {
private:
   ...
   Customer (const Customer& rhs);
   Customer& operator= (const Customer& rhs);
   ...
private:
   std::string name;
};

Customer::Customer(const Customer& rhs)
: name(rhs.name)                // copy rhs's data
{
   logCall("Customer copy constructor");
}
Customer& Customer::operator=(const Customer& rhs)
{
   logCall("Customer copy assignment operator");
   name = rhs.name;            // copy rhs's data
   return *this;
}
```
Everything here looks fine, and in fact everything is fine — untill another data member is added to Customer:
```C++
class Date { ... };            // for dates in times
class Customer {
public:
   ...
private:
   std::string name;
   Date lastTransaction;
};
```
At this point, the existing copying functions are performing a _partial copy_: they're copying the customer's `name`, but not its `lastTransaction`.

One of the most insidious ways this issue can arise is through inheritance. COnsider:
```C++
class PriorityCustomer: public Customer {
public:
   ...
   PriorityCustomer(const PriorityCustomer& rhs);
   PriorityCustomer& operator= (const PriorityCustomer& rhs);
   ...
private:
   int priority;
};

PriorityCustomer::PriorityCustomer(const PriorityCustomer& rhs)
: priority(rhs.priority)
{
   logCall("PriorityCustomer copy constructor");
}

PriorityCustomer& PriorityCustomer::operator= (const PriorityCustomer &rhs)
{
   logCall("PriorityCustomer copy assignment operator");
   priority = rhs.priority;
   return *this;
}
```
`PriorityCustomer`’s copying functions look like they’re copying everything in `PriorityCustomer` , but look again. Yes, they copy the data member that `PriorityCustomer` declares, but every `PriorityCustomer` also contains a copy of the data members it inherits from `Customer` , and those data members are not being copied at all! `PriorityCustomer`’s copy constructor specifies no arguments to be passed to its base class constructor (i.e., it makes no mention of `Customer` on its member initialization list), so the `Customer` part of the `PriorityCustomer` object will be
initialized by the `Customer` constructor taking no arguments — by the default constructor. (Assuming it has one. If not, the code won’t compile.) That constructor will perform a default initialization for `name` and `lastTransaction`.

Any time you take it upon yourself to write copying functions for a derived class, you must take care to also copy the base class parts. Those parts are typically private, of course, so you can't access them directly. Instead, derived class copying functions must invoke their corresponding base class functions:
```C++
PriorityCustomer::PriorityCustomer(const PriorityCustomer& rhs)
: Customer(rhs),                // invoke base class copy ctor
   priority(rhs.priority)
{
   logCall("PriorityCustomer copy constructor");
}
PriorityCustomer& PriorityCustomer::operator=(const PriorityCustomer& rhs)
{
   logCall("PriorityCustomer copy assignment operator");
   Customer::operator=(rhs);         // assign base class parts
   priority = rhs.priority;
   return *this;
}
```
The meaning of "copy all parts" in this Item's title should now be clear. When you're writing a copy function, be sure to
* copy all local data members and
* invoke the appropriate copying function in all base classes, too.

**Note:** If you find that your copy constructor and copy assignment operator have similar code bodies, eliminate the duplication by creating a third member function that both call. Such a function is typically private and is often named `init`. This strategy is safe, proven way to eliminate code duplication in copy constructor and copy assignment operators.

**Things to Remember**
* Copying functions should be sure to copy all of an object's data members and all of its base class parts.
* Don't try to implement one of the copying functions in terms of the other. Instead, put common functionality in a third function that both call.



