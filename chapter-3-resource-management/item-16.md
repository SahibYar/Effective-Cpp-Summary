### Item 16 - Use the same form in corresponding uses of `new` and `delete`
What's wrong with this picture?
```C++
std::string *stringArray = new std::string[100];
...
delete stringArray;
```
Here, the `new` is matched with a `delete`, Still, something is quite wrong. 99 of the 100 `string` objects pointer to by `stringArray` are unlikely to be properly destroyed, because their destructors will probably never be called.

**Background process of `new`:** When you employ a _new expression_ (i.e., dynamic creation of an object via a use of `new`), two things happen.
* memory is allocated (via function named operator `new`)
* One or more constructors are called for that memory.

**Background process of `delete`:** When you employ a _delete expression_ (i.e., use `delete`), two things happen
* one or more destructors are called for the memory.
* memory is deallocated (via function named operator `delete`)

**Example:**
```C++
std::string *stringPtr1 = new std::string;
std::string *stringPtr2 = new std::string[100];
...
delete stringPtr1;        // delete an object
delete[] stringPtr2;      // delete an array of objects
```
The rule is simple: if you use `[]` in a `new` expression, you must use `[]` in the corresponding `delete` expression. If you don't use `[]` in a `new` expression, don't use `[]` in the matching `delete` expression.

This rule is also noteworthy for the `typedef`-inclined, because it means that a `typedef`'s author must document which form of `delete` should be employed when  `new` is used to conjure up objects of the `typedef` type. For example, consider this `typedef`:
```C++
// a person's address has 4 lines, each of which is a string
typedef std::string AddressLines[4];
```
Because `AddressLines` is an array, this use of `new`,
```C++
// note that "new AddressLines" returns a string*, just like "new string[4]" would
std::string* pal = new AddressLines;
```
must be matched with the _array_ form of `delete`
```C++
delete pal;                // undefined!
delete[] pal;              // fine
```

**Things to Remember**
* if you use[] in a `new` expression, you must use `[]` in the corresponding `delete` expression. If you don't use `[]` in a `new` expression, you must not use `[]` in the corresponding `delete` expression.




