### Item 17 - Store `new`ed objects in smart pointers in standalone statements.
Suppose we have a function to reveal our processing priority and a second function to do some processing on a dynamically allocated `Widget` in accord with a priority:
```C++
int priority();
void processWidget(shared_ptr<Widget> pw, int priority);
```
`processWidget` uses a smart pointer (here, a `shared_ptr`) for dynamically allocated `Widget` it processes.
Consider now a call to `processWidget`
```C++
processWidget(new Widget, priority());
```
It won't compile because `shared_ptr`'s constructor taking a raw pointer is _explicit_, so there's no _implicit_ conversion from the raw pointer returned by expression "new Widget" to the `shared_ptr` required by `processWidget`. The following code, however, will compile:
```C++
processWidget(shared_ptr<Widget>(new Widget), priority());
```
**Note:** This may leak resources, It's illuminating to see how.
Before compilers can generate a call to `processWidget`, they have to evaluate the arguments being passed as its parameters. The second argument is just a call to the function `priority`, but the first argument consists of two parts:
* Execution of the expression `new Widget`.
* A call to the `shared_ptr` constructor.
Before `processWidget` can be called, then, compilers must generate code to do these three things:
* Call `priority`.
* Execute `new Widget`.
* Call the `shared_ptr` constructor.

In Java and C#, function parameters are always evaluated in particular order, but in C++, compilers determine the order in which these things are to be done. The `new Widget` expression must be executed before the `shared_ptr` constructor can be called, because the result of the expression is passed as an argument to the `shared_ptr` constructor, but the call to `priority` can be performed first, second or third. If compilers choose to perform it second, we end up with this sequence of operations
1. Execute `new Widget`.
2. Call `priority`.
3. Call the `shared_ptr` constructor.

But consider what will happen if the call to `priority` yields an exception. In that case, the pointer return from `new Widget` will be lost, because it won't have been stored in the `shared_ptr` would guard against resource leaks. A leak in the call to `processWidget` can arise because an exception can intervene between the time a resource is created (via `new Widget`) and time that resource is turned over to a resource-managing object.

The way to avoid problem like this is simple: use a separate statement to create the `Widget` and store it in a smart pointer, then pass the smart pointer to `processWidget`
```C++
//store newed object in a smart pointer in a standalone statement.
shared_ptr<Widget> pw(new Widget);

processWidget(pw, priority());        // this call won't leak
```
**Things to Remember**
* Store newed objects in smart pointers in standalone statements. Failure to do this can lead to subtle resource leaks when exceptions are thrown.
