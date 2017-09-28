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
When the `vector` `v` is destroyed, it is responsible for destroying all the `Widget` objects it contain. Suppose `v` has 10