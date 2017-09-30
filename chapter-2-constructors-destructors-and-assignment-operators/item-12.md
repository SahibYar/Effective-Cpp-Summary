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



