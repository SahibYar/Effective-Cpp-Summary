### Item 7 - Declare destructors virtual in polymorphic base classes.
There are alot of ways to keep track of time, so it would be reasonable to create a `TimeKeeper` base class along with derived classes for different apporaches to timekeeping:
```C++
class TimeKeeper {
public:
    TimeKeeper();
    ~TimeKeeper();
    ...
};

class AtomicClock: public TimeKeeper {...};
class WaterClock: public TimeKeeper {...};
class WristWatch: public TimeKeeper {...};
```
Many clients will want access to the time without worrying about the details of how it's calculated, 
So a **_factory function_**:
> a function that returns a base class pointer to a newly-created derived class object

can be used to return a pointer to a timekeeping object:
```C++
TimeKeeper* getTimeKeeper();        // returns a pointer to a dynamically allocated
                                    // object of a class derived from TimeKeeper
```
