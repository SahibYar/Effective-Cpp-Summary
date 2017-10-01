### Item 6 - Explicitly disallow the use of compiler-generated functions you do not want.
Real estate agents sell houses, and a software system supporting such agents would naturally have a class representing homes for sales:
```C++
class HomeForSale{...};
```
As every property is unique — no two are exactly alike. Thus we need to stop copying objects.
```C++
HomeForSale h1;
HomeForSale h2;
HomeForSale h3(h1);            // attempt to copy h1 — should not compile!
h1 = h2;                       // attempt to copy h2 — should not compile!
```
Usually, if you don't want a class to support a particular kind of functionality, you simply don't declare the function that would provide it. This strategy doesn't work for the copy constructor and copy assignment operator, because, we know, if you don't declare them and somebody tries to call them, compilers declare them for you.

The key to the solution is that all the compiler generated functions are public. To prevent these functions, from being generated, you must declare them yourself _privately_. By declaring a member function explicitly, you prevent compilers from generating their own version, and by making the function `private`, you keep people from calling it.

Mostly, the scheme isn't foolproof, because member and friend functions can still call your private functions. Unless you are clever enough not to _define_ them. Then if somebody inadvertently calls one, they'll get an error at link-time. This trick — declaring member functions `private` and deliberately not implementing them — is so well established, it's used to prevent copying in several classes in C++'s `iostream` library. Take a look, for example, at the definitions of `ios_base`, `basic_ios`, and sentry in your standard library implementation.

Applying the trick to HomeForSale is easy:
```C++
class HomeForSale {
public:
    ...
    HomeForSale(const HomeForSale&);                //declarations only
    HomeForSale& operator=(const HomeForSale&);
};
```
Now if you will try to copy `HomeForSale` objects via member or a friend functions, the linker will complain.
It's possible to move the link-time error up to compile time (always a good thing — earlier error detection is better than later) by declaring the copy constructor and copy assignment operator private not in `HomeForSale` itself, but in a base class specifically designed to prevent copying. The base class is simplicity itself:
```C++
class Uncopyable {
protected:                                // allow construction and destruction of derived objects.
    Uncopyable(){};
    ~Uncopyable(){};
private:
    Uncopyable(const Uncopyable&);        // ...but prevent copying
    Uncopyable& operator=(const Uncopyable&);
};
```
To keep `HomeForSale` objects from being copied, all we have to do now is inherit from `Uncopyable`:
```C++
class HomeForSale : private Uncopyable {    // class no longer declares copy ctor or copy assignment operator.
    ...
};
```
This works, because compilers will try to generate a copy constructor and a copy assignment operator if anybody — even a member or friend function — tries to copy a `HomeForSale` object. The compiler-generated versions of these functions will try to call their base class counterparts, and those calls will be rejected, because the copying operations are private in base class.
The implementation and use of `Uncopyable` include some subtleties, such as the fact that inheritance from `Uncopyable` needn't be public and that `Uncopyable`'s destructor need not be virtual. You can also use the version available at `Boost`. That class is named `noncopyable`.It's a fine class.

**Things to Remember:**
* To disallow functionality automatically provided by compilers, declare the corresponding member functions `private` and give no implementations. Using a base class like `Uncopyable` is one way to do this.








