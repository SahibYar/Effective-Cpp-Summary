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

#### For Example:
Suppose, working with a class for database connections:
```C++
class DBConnection {
public:
    ...
    static DBConnection create();            // function to return DBConnection objects; params omitted for simplicity
    void close();                            // close connection; throws an exception if closing fails.
};
```
To ensure that clients don't forget to call close on `DBConnection` objects, a reasonable idea would be to create a resource-managing class for `DBConnection` thats calls `close` in its destructor.
```C++
class DBConn{                                // class to manage DBConnection objects
public:
    ...
    ~DBConn()                                // make sure database connections are always closed
    {
        db.close();
    }
private:
    DBConnection db;
};
```
That allows clients to program like this:
```C++
{                                            // open a block
    DBConn dbc(DBConnection::create());      // create DBConnection object and turn it over to a DBConn object to manage.
    ...                                      // use the DBConnection object via DBConn interface.
}                                            // at the end of block, the DBConn object is destroyed,
                                             // thus automatically calling close on the DBConnection object.
```
This is fine as long as the call to `close` succeeds, but if the call yields an exception, `DBConn`'s destructor will propagate that exception, i.e., allow it to leave the destructor. That's a problem, because destructors that throw mean trouble.

There are two primary ways to avoid the trouble. `DBConn`'s destructor could:
* **Terminate the program** if `close` throws, typically by calling `abort`:
```C++
DBConn::~DBConn()
{
    try { db.close(); }
    catch(...) {
        // make a log entry that the call to close failed;
        std::abort();
    }
}
```
This is reasonable option if the program cannot continue to run after an error is encountered during destruction. It has the advantage that if allowing the exception to propagate from the destructor would lead to undefined behaviour, this prevents that from happening. That is, calling `abort` may forestall undefined behaviour.
* **Swallow the exception** arising from the call to `close`:
```C++
DBConn::~DBConn()
{
    try { db.close(); }
    catch(...) {
        // make a log entry that the call to close failed;
    }
}
```
In general, swallowing exception is a bad idea, because it supresses important information — _something failed!_ Sometimes, however, swallowing exceptions is preferable to running the risk of premature program termination or undefined behavior. For this to be a viable option, the program must be able to reliably continue execution even after an error has been encountered and ignored.

A better strategy is to design `DBConn`’s interface so that its clients have an opportunity to react to problems that may arise. 
#### For example:
`DBConn` could offer a `close` function itself, thus giving clients a chance to handle exceptions arising from that operation. It could also keep
track of whether its `DBConnection` had been closed, closing it itself in
the destructor if not. That would prevent a connection from leaking. If
the call to close were to fail in the `DBConn` destructor, however, we’d be
back to terminating or swallowing:
```C++
class DBConn {
public:
    ...
    void close()                        // new function for client use
    {
        db.close();
        closed = true;
    }
    ~DBConn()
    {
        if (!closed) {
            try {                       // close the connection if the client didn't
                db.close();
            }
            catch(...){                // if closing fails
                make log entry that call to close failed;
                ...                    // terminate or swallow
            }
        }
    }
private:
    DBConnection db;
    bool closed;
};
```
If an operation may fail by throwing an exception and there may be a need to handle that exception.
> The exception has to come from some non-destructor function.

That's because destructors that emit exceptions are dangerous, always running the risk of premature program termination or undefined behavior. In this example, telling clients to call `close` themselves doesn’t impose a burden on them; it gives them an opportunity to deal with errors they would otherwise have no chance to react to. If they don’t find that opportunity useful (perhaps because they believe that no error will really occur), they can ignore it, relying on `DBConn`’s destructor to call `close` for them. If an error occurs at that point — if `close` does throw — they’re in no position to complain if `DBConn` swallows the exception or terminates the program.



