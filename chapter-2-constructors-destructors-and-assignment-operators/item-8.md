### Item 8 - Prevent exceptions from leaving destructors.
C++ doesn't prohibit destructors from emitting exceptions, but it certainly discourage the practice. With good reason. Consider:
```C++
class Widget{
public:
    ...
    ~Widget(){...}                // assume this might emit an exception
};
void doSomething()
{
    std::vector<Widget> v;
}                                // v is automatically destroyed here.
```
When the `vector` `v` is destroyed, it is responsible for destroying all the `Widget` objects it contain. Suppose `v` has ten `Widget` objects in it, and during destruction of the first one, an exception is thrown. The other nine objects still have to be destroyed (otherwise any resources they hold would be leaked), so `v` should invoke their destructors. But suppose that during those calls, a second `Widget` object's destructor throws an exception. Now there are two simulataneously active exceptions, and that's too many for C++. Depending in the precise conditions under which such pairs of simultaneously active exceptions arise, program execution either terminates or yields undefined behaviour. In this example it yields undefined behaviour.
> C++ does not like destructors that emit exceptions!

#####For Example:
Suppose, working with a class for database connections:
```C++
class DBConnection {
public:
    ...
    static DBConnection create();            // function to return DBConnection objects; params omitted for simplicity
    void close();                            // close connection; throws an exception if closing fails.
};
```




