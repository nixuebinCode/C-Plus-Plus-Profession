# 1. Deducing Types
## Item 1: Understand template type deduction

Think of a function template as looking like this:

```c++
template<typename T>
void f(ParamType param);

f(expr); // call f with some expression
```

During compilation, compilers use `expr` to deduce two types: one for `T` and one for `ParamType`. These types are frequently different, because `ParamType` often contains adornments, e.g., `const` or reference qualifiers:

```c++
template<typename T>
void f(const T& param); // ParamType is const T&
int x = 0;
f(x); // call f with an int
```

`T` is deduced to be `int`, but `ParamType` is deduced to be `const int&`.

It’s natural to expect that the type deduced for `T` is the same as the type of the argument passed to the function, i.e., that `T` is the type of `expr`. In the above example, that’s the case: `x` is an `int`, and `T` is deduced to be int. But it doesn’t always work that way. The type deduced for `T` is dependent not just on the type of `expr`, but also on the form of `ParamType`. There are three cases:

### Case 1: `ParamType` is a Reference or Pointer, but not a Universal Reference
Type deduction works like this:

* If `expr`’s type is a reference, ignore the reference part.
* Then pattern-match `expr`’s type against `ParamType` to determine `T`.

```c++
template<typename T>
void f(T& param); // param is a reference

int x = 27; // x is an int
const int cx = x; // cx is a const int
const int& rx = x; // rx is a reference to x as a const int

f(x); // T is int, param's type is int&
f(cx); // T is const int, param's type is const int&
f(rx); // T is const int, param's type is const int&
```

Passing a `const` object to a template taking a `T&` parameter is safe: the constness of the object becomes part of the type deduced for `T`.

In the third example, note that even though `rx`’s type is a reference, `T` is deduced to be a non-reference. That’s because `rx`’s reference-ness is ignored during type deduction.

If we change the type of `f`’s parameter from `T&` to `const T&`, things change a little, because we’re now assuming that param is a reference-to-const, there’s no longer a need for `const` to be deduced as part of  `T`:

```c++
template<typename T>
void f(const T& param); // param is now a ref-to-const

int x = 27; // as before
const int cx = x; // as before
const int& rx = x; // as before

f(x); // T is int, param's type is const int&
f(cx); // T is int, param's type is const int&
f(rx); // T is int, param's type is const int&
```

If param were a pointer (or a pointer to const) instead of a reference, things would work essentially the same way:

```c++
template<typename T>
void f(T* param); // param is now a pointer

int x = 27; // as before
const int *px = &x; // px is a ptr to x as a const int

f(&x); // T is int, param's type is int*
f(px); // T is const int, param's type is const int*
```

### Case 2: `ParamType` is a Universal Reference

Universal reference parameters are declared like rvalue references( T&& ), they behave differently when lvalue arguments are passed in:

* If `expr` is an lvalue, both `T` and `ParamType` are deduced to be lvalue references. That’s doubly unusual. First, it’s the only situation in template type deduction where `T` is deduced to be a reference. Second, although `ParamType` is declared using the syntax for an rvalue reference, its deduced type is an lvalue reference.
* If `expr` is an rvalue, the “normal” (i.e., Case 1) rules apply

```c++
template<typename T>
void f(T&& param); // param is now a universal reference

int x = 27; // as before
const int cx = x; // as before
const int& rx = x; // as before

f(x); // x is lvalue, so T is int&, param's type is also int&
f(cx); // cx is lvalue, so T is const int&, param's type is also const int&
f(rx); // rx is lvalue, so T is const int&, param's type is also const int&
f(27); // 27 is rvalue, so T is int, param's type is therefore int&&
```

### Case 3: `ParamType` is Neither a Pointer nor a Reference

```c++
template<typename T>
void f(T param); // param is now passed by value
```

That means that param will be a copy of whatever is passed in—a completely new object. The fact that param will be a new object motivates the rules that govern how `T` is deduced from `expr`:

* As before, if `expr`’s type is a reference, ignore the reference part
* If, after ignoring `expr`’s reference-ness, `expr` is const, ignore that, too. 

```c++
int x = 27; // as before
const int cx = x; // as before
const int& rx = x; // as before

f(x); // T's and param's types are both int
f(cx); // T's and param's types are again both int
f(rx); // T's and param's types are still both int
```

Note that even though `cx` and `rx` represent const values, param isn’t const. That makes sense. param is an object that’s completely independent of `cx` and `rx`—a copy of `cx` or `rx`. The fact that `cx` and `rx` can’t be modified says nothing about whether param can be. That’s why `expr`’s constness is ignored.

And consider the case where `expr` is a const pointer to a const object, and `expr` is passed to a by-value `param`:

```c++
template<typename T>
void f(T param); // param is still passed by value
const char* const ptr = f(ptr); // ptr is const pointer to const object
 								// pass arg of type const char * const
```

When `ptr` is passed to `f`, the bits making up the pointer are copied into param. As such, **the pointer itself (`ptr`) will be passed by value**. In accord with the type deduction rule for by-value parameters, the constness of `ptr` will be ignored, and the type deduced for param will be const char*

### Array Arguments

Array types are different from pointer types, even though they sometimes seem to be interchangeable:

#### Pass by value

```c++
const char name[] = "J. P. Briggs"; // name's type is const char[13]

template<typename T>
void f(T param); // template with by-value parameter

f(name);
```

Remember that as a function parameter, the array declaration is treated as a pointer declaration:

```c++
void myFunc(int param[]);
void myFunc(int* param); // same function as above
```

Because array parameter declarations are treated as if they were pointer parameters, the type of an array that’s passed to a template function **by value** is deduced to be a pointer type:

```c++
f(name); // name is array, but T deduced as const char*
```

#### Pass by reference

Although functions can’t declare parameters that are truly arrays, they can declare parameters that are references to arrays! So if we modify the template `f` to take its argument by reference:

```c++
template<typename T>
void f(T& param); // template with by-reference parameter

f(name); // pass array to f
```

The type deduced for `T` is the actual type of the array! That type includes the size of the array, so in this example, `T` is deduced to be `const char [13]`, and the type of `f`’s parameter (a reference to this array) is `const char (&)[13]`

Interestingly, the ability to declare references to arrays enables creation of a template that deduces the number of elements that an array contains:

```c++
template<typename T, std::size_t N>
constexpr std::size_t arraSize(T (&)[N]) noexcept{
    return N;
}
```

### Function Arguments

Arrays aren’t the only things in C++ that can decay into pointers. Function types can decay into function pointers:

```c++
void someFunc(int, double); // someFunc is a function; type is void(int, double)

template<typename T>
void f1(T param); // in f1, param passed by value

template<typename T>
void f2(T& param); // in f2, param passed by ref

f1(someFunc); // param deduced as ptr-to-func; type is void (*)(int, double)
f2(someFunc); // param deduced as ref-to-func; type is void (&)(int, double
```

### Things to Remember

* During template type deduction, arguments that are references are treated as non-references, i.e., their reference-ness is ignored.
* When deducing types for universal reference parameters, lvalue arguments get special treatment.
* When deducing types for by-value parameters, const arguments are treated as non-const.
* During template type deduction, arguments that are array or function names decay to pointers, unless they’re used to initialize references.

## Item 2: Understand `auto` type deduction

Deducing types for `auto` is, with only one exception, the same as deducing types for templates.

In Item 1, template type deduction is explained using this general function template：

```C++ 
template<typename T>
void f(ParamType param);

f(expr); // call f with some expression
```


In the call to `f`, compilers use `expr` to deduce types for `T` and `ParamType`.

When a variable is declared using `auto`, `auto` plays the role of `T` in the template, and the type specifier for the variable acts as `ParamType`.

```c++
auto x = 27;		// the type specifier for x is simply auto by itself
const auto cx = x;	// the type specifier is const auto
const auto& rx = x;	// the type specifier is const auto&
```

To deduce types for `x`, `cx`, and `rx` in these examples, compilers act as if there were a template for each declaration as well as a call to that template with the corresponding initializing expression:

```c++
template<typename T> 
void func_for_x(T param);
func_for_x(27);

template<typename T> 
void func_for_cx(const T param);
func_for_cx(x);

template<typename T>
void func_for_rx(const T& param);
func_for_rx(x);
```

As type deduced in template, in a variable declaration using `auto`, the type specifier takes the place of `ParamType`, so there are three cases for that, too:

* The type specifier is a pointer or reference, but not a universal reference
* The type specifier is a universal reference
* The type specifier is neither a pointer nor a reference

```c++
auto x = 27; 			// case 3 (x is neither ptr nor reference)
const auto cx = x; 		// case 3 (cx isn't either)
const auto& rx = x; 	// case 1 (rx is a non-universal ref.)

auto&& uref1 = x; 		// x is int and lvalue,
						// so uref1's type is int&
auto&& uref2 = cx; 		// cx is const int and lvalue,
						// so uref2's type is const int&
auto&& uref3 = 27; 		// 27 is int and rvalue,
						// so uref3's type is int&&
```

Array and function names decay into pointers for non-reference type specifiers. That happens in auto type deduction, too:

```c++
const char name[] = "R. N. Briggs"; // name's type is const char[13]

auto arr1 = name; // arr1's type is const char*
auto& arr2 = name; // arr2's type is const char (&)[13]

void someFunc(int, double); // someFunc is a function; type is void(int, double)
auto func1 = someFunc; // func1's type is void (*)(int, double)
auto& func2 = someFunc; // func2's type is void (&)(int, double)
```

### Special type deduction rule for `auto`

If you want to declare `an` int with an initial value of 27, you have the following four choice:

```c++
// four syntaxes, but only one result: an int with value 27
int x1 = 27;
int x2(27);
int x3 = { 27 };
int x4{ 27 };
```

But if you substitute `int` with `auto`, things will change:

```c++
auto x1 = 27;		// type is int, value is 27
auto x2(27);		// ditto
auto x3 = { 27 };	// type is std::initializer_list<int>, value is { 27 }
auto x4{ 27 };		// ditto
```

The first two statements do, indeed, declare a variable of type `int` with value 27. The second two, however, declare a variable of type `std::initializer_list<int>` containing a single element with value 27. 

When the initializer for an auto-declared variable is enclosed in braces, the deduced type is a `std::initializer_list`.

The treatment of braced initializers is the only way in which `auto` type deduction and template type deduction differ:

```c++
auto x = { 11, 23, 9 }; // x's type is std::initializer_list<int>

template<typename T> 	// template with parameter declaration equivalent to x's declaration
void f(T param); 
f({ 11, 23, 9 }); 		// error! can't deduce type for T
```

However, if you specify in the template that param is a `std::initializer_list<T>` for some unknown `T`, template type deduction will deduce what `T` is:

```c++
template<typename T>
void f(std::initializer_list<T> initList);

f({ 11, 23, 9 }); 		// T deduced as int
```

### `auto` in a function return type or a lambda parameter implies template type deduction, not auto type deduction

C++14 permits `auto` to indicate that a function’s return type should be deduced, and C++14 lambdas may use `auto` in parameter declarations.

```c++
auto createInitList()
{
	return { 1, 2, 3 }; // error: can't deduce type
}

std::vector<int> v;
auto resetV = [&v](const auto& newValue) { v = newValue; }; // C++14
resetV({ 1, 2, 3 }); // error! can't deduce type // for { 1, 2, 3 }
```

## Item 3: Understand `decltype`

Given a name or an expression, `decltype` tells you the name’s or the expression’s type.

### The typical cases for `decltype`

In contrast to what happens during type deduction for templates and `auto`, `decltype` typically parrots back the exact type of the name or expression you give it:

```c++
const int i = 0; 						// decltype(i) is const int
bool f(const Widget& w); 				// decltype(w) is const Widget&
										// decltype(f) is bool(const Widget&)
struct Point {
int x, y; 								// decltype(Point::x) is int
}; 										// decltype(Point::y) is int
Widget w; 								// decltype(w) is Widget
if (f(w)) … 							// decltype(f(w)) is bool
vector<int> v; 							// decltype(v) is vector<int>
if (v[0] == 0)							// decltype(v[0]) is int&
```

### The use of `decltype` to compute the return type

The type returned by a container’s `operator[]` depends on the container. `decltype` makes it easy to express that.

```c++
template<typename Container, typename Index> // works, but requires refinement
auto authAndAccess(Container& c, Index i) -> decltype(c[i])
{
	authenticateUser();
	return c[i];
}
```

The use of `auto` before the function name has nothing to do with type deduction. it indicates that C++11’s `trailing return type` syntax is being used. Trailing return type has the advantage that **the function’s parameters can be used in the specification of the return type.**

in C++14 we can omit the trailing return type, leaving just the leading `auto`. With that form of declaration, `auto` does mean that type deduction will take place:

```c++
template<typename Container, typename Index> 	// C++14;
auto authAndAccess(Container& c, Index i) 		// not quite correct
{
	authenticateUser();
	return c[i]; // return type deduced from c[i]
}
```

For functions with an `auto` return type specification, compilers employ template type deduction. In this case, that’s problematic. **`operator[]` returns a `T&`, but during template type deduction, the reference-ness of an initializing expression is ignored.**

### `decltype(auto)`

To fix this issue, in C++14 we can use the `decltype(auto)` specifier. `auto` specifies that the type is to be deduced, and `decltype` says that `decltype` rules should be used during the deduction:

```c++
template<typename Container, typename Index> 	// C++14; works,
decltype(auto) 									// but still requires refinement
authAndAccess(Container& c, Index i)
{
	authenticateUser();
	return c[i];
}
```

Now `authAndAccess` will truly return whatever `c[i]` returns.

`decltype(auto)` can also be convenient for declaring variables when you want to apply the `decltype` type deduction rules to the initializing expression:

```c++
Widget w;
const Widget& cw = w;
auto myWidget1 = cw; // auto type deduction: myWidget1's type is Widget
decltype(auto) myWidget2 = cw; // decltype type deduction: myWidget2's type is const Widget&
```

### The refinement to `authAndAccess`

Look again at the declaration for the C++14 version of `authAndAccess`:

```c++
template<typename Container, typename Index>
decltype(auto) authAndAccess(Container& c, Index i);
```

The container is passed by lvalue-reference-to-non-const, this means it’s not possible to pass rvalue containers to this function. Rvalues can’t bind to lvalue references.

Have `authAndAccess` employ a reference parameter that can bind to lvalues and rvalues -- universal references.

```c++
template<typename Container, typename Index> // c is now a universal reference
decltype(auto) authAndAccess(Container&& c, Index i);
```

And we need to update the template’s implementation to bring it into accord with Item 25’s admonition to apply `std::forward` to universal references:

```c++
template<typename Container, typename Index> // final C++14 version
decltype(auto)
authAndAccess(Container&& c, Index i)
{
	authenticateUser();
	return std::forward<Container>(c)[i];
}
```

### A few special cases for `decltype`

if an lvalue expression other than a name has type `T`, decltype reports that type as `T&`:

```c++
int x = 0;
```

`x` is the name of a variable, so `decltype(x`) is `int`.

But wrapping the name `x` in parentheses—“`(x)`”—yields an expression more complicated than a name. Being a name, `x` is an lvalue, and C++ defines the expression `(x)` to be an lvalue, too. `decltype((x))` is therefore `int&`.

## Item 4: Know how to view deduced types

We’ll explore three possibilities: getting type deduction information as you edit your code, getting it during compilation, and getting it at runtime.

### IDE Editors

Code editors in IDEs often show the types of program entities when you do something like hover your cursor over the entity.

```c++
const int theAnswer = 42;
auto x = theAnswer;
auto y = &theAnswer;
```

An IDE editor would likely show that `x`’s deduced type was `int` and `y`’s was `const int*`.

For simple types like `int`, information from IDEs is generally fine. However, when more complicated types are involved, the information displayed by IDEs may not be particularly helpful.

### Compiler Diagnostics

An effective way to get a compiler to show a type it has deduced is to use that type in a way that leads to compilation problems. **The error message** reporting the problem is virtually sure to mention the type that’s causing it.

For example, we first declare a class template that we don’t define. Something like this does nicely:

```c++
template<typename T> 	// declaration only for TD;
class TD; 				// TD == "Type Displayer"
```

Attempts to instantiate this template will elicit an error message, because there’s no template definition to instantiate. To see the types for `x` and `y`, just try to instantiate `TD` with their types:

```c++
TD<decltype(x)> xType; // elicit errors containing
TD<decltype(y)> yType; // x's and y's types
```

You will see the following error message:

```c++
error: aggregate 'TD<int> xType' has incomplete type and cannot be defined
     TD<decltype(x)> xType;
error: aggregate 'TD<const int*> yType' has incomplete type and cannot be defined
     TD<decltype(y)> yType;
```

### Runtime Output

In our continuing quest to see the types deduced for `x` and `y`, you may figure we can write this:

```c++
std::cout << typeid(x).name() << '\n'; // display types for
std::cout << typeid(y).name() << '\n'; // x and y
```

This approach relies on the fact that invoking `typeid` on an object such as `x` or `y` yields a `std::type_info` object, and `std::type_info` has a member function, `name`, that produces a C-style string (i.e., a const char*) representation of the name of the type.

Consider a more complex example: 

```c++
template<typename T> 		// template function to
void f(const T& param); 	// be called

std::vector<Widget> createVec(); // factory function

const auto vw = createVec();
if (!vw.empty()) {
	f(&vw[0]); // call f
}
```

Suppose now we want to know what types are inferred for the template type parameter `T` and the function parameter `param` in f.

Loosing `typeid` on the problem is straightforward:

```c++
template<typename T>
void f(const T& param)
{
	using std::cout;
    cout << "T = " << typeid(T).name() << '\n';
	cout << "param = " << typeid(param).name() << '\n';
}
```

Executables produced by the GNU and Clang compilers produce this output:

```c++
T = PK6Widget
param = PK6Widget
```

So these compilers tell us that both `T` and `param` are of type `const Widget*`. Sadly, the results of `std::type_info::name` are not reliable. In this case, the type that compilers report for `param` are incorrect.

The specification for `std::type_info::name` mandates that the type be treated as if it had been passed to a template function as a by-value parameter. That means that if the type is a reference, its reference-ness is ignored, and if the type after reference removal is const, its constness is also ignored. That’s why param’s type—which is `const Widget * const &`—is reported as `const Widget*`.

Where `std::type_info::name` and IDEs may fail, the **Boost TypeIndex library** (often written as Boost.TypeIndex) is designed to succeed.

```c++
#include <boost/type_index.hpp>
template<typename T>
void f(const T& param)
{
	using std::cout;
	using boost::typeindex::type_id_with_cvr;
	// show T
	cout << "T = " << type_id_with_cvr<T>().pretty_name() << '\n';
	// show param's type
	cout << "param = " << type_id_with_cvr<decltype(param)>().pretty_name() << '\n';
	…
}
```

Under compilers from GNU and Clang, Boost.TypeIndex produces this (accurate) output:

```c++
T = Widget const*
param = Widget const* const&
```

# 2. `auto`

## Item 5: Prefer `auto` to explicit type declarations

### Avoidance of uninitialized variables

`auto` variables have their type deduced from their initializer, so they must be initialized. That means you can wave goodbye to a host of uninitialized variable problems:

```c++
int x1; 		// potentially uninitialized
auto x2; 		// error! initializer required
auto x3 = 0; 	// fine, x's value is well-defined
```

### Avoidance of verbose variable declarations, and the ability to directly hold closures
Because auto uses type deduction, it can represent types known only to compilers:

```c++
auto derefUPLess = // comparison func. for Widgets pointed to by std::unique_ptrs
	[](const std::unique_ptr<Widget>& p1, 
		const std::unique_ptr<Widget>& p2)
	{ return *p1 < *p2; };
```

In C++14, parameters to lambda expressions may involve `auto`:

```c++
auto derefLess = // C++14 comparison function for values pointed to by anything pointer-like
	[](const auto& p1, 
   		const auto& p2) 
	{ return *p1 < *p2; };
```

Without `auto`, to declare the `derefLess`, you might use the `std::function`. `std::function` is a template in the C++11 Standard Library that generalizes the idea of a function pointer. `std::function` objects can refer to any callable object：

```c++
std::function<bool(const std::unique_ptr<Widget>&, const std::unique_ptr<Widget>&)> func;
```

Because lambda expressions yield callable objects, closures can be stored in `std::function` objects. So without `auto` , you can declare `derefLess` like this:

```c++
std::function<bool (const std::unique_ptr<Widget>& p1, 
					const std::unique_ptr<Widget>& p2)> 
   	derefLess = [](const std::unique_ptr<Widget>& p1, 
					const std::unique_ptr<Widget>& p2)
				{ return *p1 < *p2; };
```

Note that using `std::function` is not the same as using `auto`：

* An `auto`-declared variable holding a closure has the same type as the closure, and as such it uses only as much memory as the closure requires.
* The type of a `std::function` declared variable holding a closure is an instantiation of the `std::function` template, and that has a fixed size for any given signature.

The result is that the `std::function` approach is generally bigger and slower than the `auto` approach, and it may yield out-of-memory exceptions, too.

### Avoidance of problems related to “type shortcuts”

```c++
std::vector<int> v;
…
unsigned sz = v.size();
```

The official return type of `v.size()` is `std::vector<int>::size_type`. And On 32-bit Windows, for example, both `unsigned` and `std::vector<int>::size_type` are the same size, but on 64-bit Windows, `unsigned` is 32 bits, while `std::vector<int>::size_type` is 64 bits. This means that code that works under 32-bit Windows may behave incorrectly under 64-bit Windows.

```c++
auto sz = v.size(); // sz's type is std::vector<int>::size_type
```

Similarly see this example:

```c++
std::unordered_map<std::string, int> m;
…
for (const std::pair<std::string, int>& p : m)
{
	… // do something with p
}
```

Remember that the key part of a `std::unordered_map` is `const`, so the type of `std::pair` in the hash table isn’t `std::pair<std::string, int>`, it’s `std::pair<const std::string, int>`. As a result, compilers will strive to find a way to convert `std::pair<const std::string, int>` objects to
`std::pair<std::string, int>` objects. They’ll succeed by creating a temporary object of the type that `p` wants to bind to by copying each object in `m`, then binding the reference `p` to that **temporary object**. At the end of each loop iteration, the temporary object will be destroyed.

```c++
for (const auto& p : m)
{
	… // as before
}	
```

### Refactorings are facilitated by the use of `auto`

Furthermore, `auto` types automatically change if the type of their initializing expression changes. For example, if a function is declared to return an `int`, but you later decide that a `long` would be better, the calling code automatically updates itself the next time you compile if the results of calling the function are stored in `auto` variables.

## Item 6: Use the explicitly typed initializer idiom when `auto` deduces undesired types
Suppose I have a function that takes a `Widget` and returns a `std::vector<bool>`, where each `bool` indicates whether the `Widget` offers a particular feature:

```c++
std::vector<bool> features(const Widget& w);
```

Further suppose that bit 5 indicates whether the `Widget` has high priority. We can thus write code like this:

```c++
Widget w;
bool highPriority = features(w)[5]; // is w high priority?
processWidget(w, highPriority); // process w in accord with its priority
```

The code above works fine. But if we use `auto` instead:

```c++
auto highPriority = features(w)[5]; // is w high priority?
processWidget(w, highPriority); // undefined behavior!
```

The call to `processWidget` now has undefined behavior. The type of `highPriority` is no longer `bool`.

Though `std::vector<bool>` conceptually holds `bool`s, `operator[]` for `std::vector<bool>` doesn’t return a reference to an element of the container. Instead, it returns an object of type `std::vector<bool>::reference` (a class nested inside `std::vector<bool>`).

`std::vector<bool>::reference` exists because `std::vector<bool>` is specified to represent its `bool`s in packed form, one bit per `bool`. `operator[]` for `std::vector<T>` is supposed to return a `T&`, but C++ forbids references to bits. Not being able to return a `bool&`, `operator[]` for `std::vector<bool>` returns an object that acts like a `bool&`. 

With this information in mind, look again at this part of the original code:

```c++
bool highPriority = features(w)[5]; // declare highPriority's type explicitly
```

Here, `operator[]` returns a `std::vector<bool>::reference` object, which is then implicitly converted to the `bool` that is needed to initialize `highPriority`



`std::vector<bool>::reference` is an example of a **proxy class**: a class that exists for the purpose of emulating and augmenting the behavior of some other type. Standard Library’s smart pointer types are proxy classes that graft resource management onto raw pointers.

Some proxy classes are designed to be apparent to clients. That’s the case for `std::shared_ptr` and `std::unique_ptr`, for example. Other proxy classes are designed to act more or less invisibly. `std::vector<bool>::reference` is an example of such “invisible” proxies.

As a general rule, **“invisible” proxy classes don’t play well with `auto`.** You therefore want to avoid code of this form:

```c++
auto someVar = expression of "invisible" proxy class type;
```

### How to find a  “invisible” proxy class

libraries using “invisible” proxy classes often document that they do so.

Where documentation comes up short, header files fill the gap. Function signatures usually reflect their existence:

```c++
namespace std { 			// from C++ Standards
template <class Allocator>
class vector<bool, Allocator> {
public:
	…
	class reference { … };
	reference operator[](size_type n);
	…
};
}
```

The unconventional return type for `operator[]` in this case is a tip-off that a proxy class is in use.

### The explicitly typed initializer idiom

The explicitly typed initializer idiom involves declaring a variable with `auto`, but casting the initialization expression to the type you want `auto` to deduce:

```c++
auto highPriority = static_cast<bool>(features(w)[5]);
```

Here, `features(w)[5]` continues to return a `std::vector<bool>::reference` object, just as it always has, but the cast changes the type of the expression to `bool`, which `auto` then deduces as the type for `highPriority`.

Applications of the idiom aren’t limited to initializers yielding proxy class types. It can also be useful to emphasize that you are deliberately creating a variable of a type that is different from that generated by the initializing expression:

```c++
double calcEpsilon(); // return tolerance value
float ep = calcEpsilon(); // impliclitly convert double → float
auto ep = static_cast<float>(calcEpsilon());
```

A declaration using the explicitly typed initializer idiom announces “I’m deliberately reducing the precision of the value returned by the function.”

# 3. Moving to Modern C++

## Item 7: Distinguish between () and {} when creating objects

As a general rule, initialization values may be specified with parentheses, an equals sign, or braces:

```c++
int x(0); 	// initializer is in parentheses
int y = 0; 	// initializer follows "="
int z{ 0 }; // initializer is in braces
int z = { 0 }; // initializer uses "=" and braces; the same as the braces-only version
```

for userdefined types, it’s important to distinguish initialization from assignment, because different function calls are involved:

```c++
Widget w1; 			// call default constructor
Widget w2 = w1; 	// not an assignment; calls copy ctor
w1 = w2; 			// an assignment; calls copy operator=
```

### uniform initialization

A single initialization syntax that can, at least in concept, be used anywhere and express everything. It’s based on braces, and for that reason I prefer the term *braced initialization*.

Using braces, specifying the initial contents of a container:

```c++
std::vector<int> v{ 1, 3, 5 }; // v's initial content is 1, 3, 5
```

Braces can also be used to specify default initialization values for non-static data members, This capability—new to C++11—is shared with the “=” initialization syntax, but not with parentheses:

```c++
class Widget {
…
private:
	int x{ 0 }; // fine, x's default value is 0
	int y = 0; // also fine
	int z(0); // error!
};
```

On the other hand, uncopyable objects (e.g., `std::atomics`—see Item 40) may be initialized using braces or parentheses, but not using “=”:

```c++
std::atomic<int> ai1{ 0 }; // fine
std::atomic<int> ai2(0); // fine
std::atomic<int> ai3 = 0; // error!
```

#### prohibit implicit narrowing conversions among built-in types

If the value of an expression in a braced initializer isn’t guaranteed to be expressible by the type of the object being initialized, the code won’t compile:

```c++
double x, y, z;
…
int sum1{ x + y + z }; 	// error! sum of doubles may
						// not be expressible as int
```

Initialization using parentheses and “=” doesn’t check for narrowing conversions

```c++
int sum2(x + y + z); 	// okay (value of expression
						// truncated to an int)
int sum3 = x + y + z; 	// ditto
```

#### immunity to C++’s most vexing parse

If you try to call a `Widget` constructor with zero arguments, you may declare a function instead of an object:

```c++
Widget w2(); 	// most vexing parse! declares a function
				// named w2 that returns a Widget!
```

Functions can’t be declared using braces for the parameter list, so defaultconstructing an object using braces doesn’t have this problem:

```c++
Widget w3{}; // calls Widget ctor with no args
```

### braced initializers, std::initializer_lists, and constructor overloading

In constructor calls, parentheses and braces have the same meaning as long as `std::initializer_list` parameters are not involved:

```c++
class Widget {
public:
	Widget(int i, bool b); 		// ctors not declaring
	Widget(int i, double d); 	// std::initializer_list params
	…
};
Widget w1(10, true); 			// calls first ctor
Widget w2{10, true}; 			// also calls first ctor
Widget w3(10, 5.0); 			// calls second ctor
Widget w4{10, 5.0}; 			// also calls second ctor
```

If, however, one or more constructors declare a parameter of type `std::initializer_list`, calls using the braced initialization syntax **strongly** prefer the overloads taking `std::initializer_lists`.

```c++
class Widget {
public:
	Widget(int i, bool b); 							// as before
	Widget(int i, double d); 						// as before
    Widget(std::initializer_list<long double> il); 	// added
    
    operator float() const; 						// convert to float
};

Widget w1(10, true); 								// uses parens and, as before,
													// calls first ctor
Widget w2{10, true}; 								// uses braces, but now calls
													// std::initializer_list ctor
													// (10 and true convert to long double)
Widget w3(10, 5.0); 								// uses parens and, as before,
													// calls second ctor
Widget w4{10, 5.0}; 								// uses braces, but now calls
													// std::initializer_list ctor
													// (10 and 5.0 convert to long double)
Widget w5(w4); 										// uses parens, calls copy ctor
Widget w6{w4}; 										// uses braces, calls std::initializer_list ctor
													// (w4 converts to float, and float
													// converts to long double)
```

Compilers’ determination to match braced initializers with constructors taking `std::initializer_lists` is so strong, it prevails even if the `std::initializer_list` constructor can’t be called：

```c++
class Widget {
public:
	Widget(int i, bool b); 		// as before
	Widget(int i, double d); 	// as before
	Widget(std::initializer_list<bool> il); // element type is now bool
};

Widget w{10, 5.0}; // error! requires narrowing conversions
```

Here, compilers will ignore the first two constructors and try to call the constructor taking a `std::initializer_list<bool>`. Calling that constructor would require narrowing conversions, which are prohibited inside braced initializers, so the call is invalid, and the code is rejected.

Only if there’s no way to convert the types of the arguments in a braced initializer to the type in a `std::initializer_list` do compilers fall back on normal overload resolution.

```c++
class Widget {
public:
	Widget(int i, bool b); // as before
	Widget(int i, double d); // as before
	// std::initializer_list element type is now std::string
    // There is no way to convert ints and bools to std::strings
	Widget(std::initializer_list<std::string> il);
};

Widget w1(10, true); 	// uses parens, still calls first ctor
Widget w2{10, true}; 	// uses braces, now calls first ctor
Widget w3(10, 5.0); 	// uses parens, still calls second ctor
Widget w4{10, 5.0}; 	// uses braces, now calls second ctor
```

### Empty brace

Suppose you use an empty set of braces to construct an object that supports default construction and also supports `std::initializer_list` construction. What do your empty braces mean? 

If they mean “no arguments,” you get default construction, but if they mean “empty `std::initializer_list`,” you get construction from a `std::initializer_list` with no elements.

The rule is that you get default construction. **Empty braces mean no arguments, not an `empty std::initializer_list`:**

```c++
class Widget {
public:
	Widget(); 								// default ctor
	Widget(std::initializer_list<int> il); 	// std::initializer_list ctor

};
Widget w1; 									// calls default ctor
Widget w2{}; 								// also calls default ctor
Widget w3(); 								// most vexing parse! declares a function!
```

If you want to call a `std::initializer_list` constructor with an empty `std::initializer_list`, you do it by making the empty braces a constructor argument：

```c++
Widget w4({}); // calls std::initializer_list ctor with empty list
Widget w5{{}}; // ditto Item
```

### Parentheses and braces for `std::vector<numeric type>`

An example of where the choice between parentheses and braces can make a significant difference is creating a `std::vector<numeric type>` with two
arguments.

`std::vector` has a non-`std::initializer_list` constructor that allows you to specify the initial size of the container and a value each of the initial elements should have, but it also has a constructor taking a `std::initializer_list` that permits you to specify the initial values in the container.

```c++
std::vector<int> v1(10, 20); 		// use non-std::initializer_list
									// ctor: create 10-element
									// std::vector, all elements have
									// value of 20
std::vector<int> v2{10, 20}; 		// use std::initializer_list ctor:
									// create 2-element std::vector,
									// element values are 10 and 20
```

### Parentheses and braces for object creation inside templates

Suppose you’d like to create an object of an arbitrary type from an arbitrary number of arguments:

```c++
template<typename T, 			// type of object to create
			typename... Ts> 	// types of arguments to use
void doSomeWork(Ts&&... params)
{
	T localObject(std::forward<Ts>(params)...); // using parens
	T localObject{std::forward<Ts>(params)...}; // using braces
	…
}

std::vector<int> v;
…
doSomeWork<std::vector<int>>(10, 20);
```

If `doSomeWork` uses parentheses when creating `localObject`, the result is a `std::vector` with 10 elements. If `doSomeWork` uses braces, the result is a `std::vector` with 2 elements. Which is correct? The author of doSomeWork can’t know. Only the caller can.

This is precisely the problem faced by the Standard Library functions `std::make_unique` and `std::make_shared`. These functions resolve the problem by internally using parentheses and by documenting this decision as part of their interfaces.

## Item 8: Prefer nullptr to 0 and NULL

Neither `0` nor `NULL` has a pointer type. 

In C++98, the primary implication of this was that overloading on pointer and integral types could lead to surprises. Passing `0` or `NULL` to such overloads never called a pointer overload:

```c++
void f(int); 	// three overloads of f
void f(bool);
void f(void*);
f(0); 			// calls f(int), not f(void*)
f(NULL); 		// might not compile, but typically calls
				// f(int). Never calls f(void*)
f(nullptr); 	// calls f(void*) overload
```

`nullptr`’s advantage is that it doesn’t have an integral type. To be honest, it doesn’t have a pointer type, either, but you can think of it as **a pointer of all types.** Calling the overloaded function `f` with `nullptr` calls the `void*` overload, because `nullptr` can’t be viewed as anything integral.

## Item 9: Prefer alias declarations to typedefs

`typedef`  and alias declarations:

```c++
typedef
	std::unique_ptr<std::unordered_map<std::string, std::string>>
	UPtrMapSS;
using UPtrMapSS =
	std::unique_ptr<std::unordered_map<std::string, std::string>>;
```

* **alias declaration is easier to swallow when dealing with types involving function pointers**

  ```c++
  // FP is a synonym for a pointer to a function taking an int and
  // a const std::string& and returning nothing
  typedef void (*FP) (int, const std::string&);	// typedef
  using FP = void (*)(int, const std::string&);	// alias declaration
  ```

* **alias declarations may be templatized (in which case they’re called *alias templates*), while typedefs cannot**

  Consider defining a synonym for a linked list that uses a custom allocator, `MyAlloc`.

  ```c++
  template<typename T> 							
  using MyAllocList = std::list<T, MyAlloc<T>>; 	
  
  template<typename T>
  struct MyAllocList { 
  	typedef std::list<T, MyAlloc<T>> type;
  };
  
  MyAllocList<Widget> lw; 		// client code
  MyAllocList<Widget>::type lw; 	// client code
  ```

  It gets worse, If you want to use the `typedef` inside a template with a template parameter, you have to precede the `typedef` name with `typename`:

  ```c++
  template<typename T>
  class Widget {
  private:
  	typename MyAllocList<T>::type list;
  	…
  };
  
  template<typename T>
  class Widget {
  private:
  	MyAllocList<T> list; // no "typename", no "::type"
  };
  ```

## Item 10: Prefer scoped `enum`s to unscoped `enum`s

As a general rule, declaring a name inside curly braces limits the visibility of that name to the scope defined by the braces. Not so for the enumerators declared in C++98-style enums:

```c++
enum Color { black, white, red }; 	// black, white, red are
									// in same scope as Color
auto white = false; 				// error! white already
									// declared in this scope
```

The fact that these enumerator names leak into the scope containing their `enum` definition gives rise to the official term for this kind of `enum`: **unscoped**.

Their new C++11 counterparts, **scoped** `enums`, don’t leak names in this way:

```c++
enum class Color { black, white, red }; 	// black, white, red
											// are scoped to Color
auto white = false; 						// fine, no other
											// "white" in scope
Color c = white;							// error! no enumerator named
											// "white" is in this scope
Color c = Color::white; 					// fine
auto c = Color::white;						// also fine (and in accord with Item 5's advice)
```

### Scoped enums are much more stongly typed

Scoped `enum`s have a second compelling advantage: their enumerators are much more strongly typed. Enumerators for unscoped enums implicitly convert to integral types:

```c++
std::vector<std::size_t> 			// func. returning prime factors of x
primeFactors(std::size_t x);

enum Color { black, white, red }; 	// unscoped enum
Color c = red;
if (c < 14.5) { 					// compare Color to double (!)
	auto factors = primeFactors(c);	// compute prime factors of a Color (!)
	…
}

enum class Color { black, white, red }; 	// enum is now scoped
Color c = Color::red; 						// as before, but with scope qualifier
if (c < 14.5) { 							// error! can't compare Color and double
    auto factors = primeFactors(c); 		// error! can't pass Color to function expecting std::size_t
	…
}

// If you honestly want to perform a conversion from Color to a different type, use the cast
if (static_cast<double>(c) < 14.5) { 		// odd code, but it's valid
	auto factors = primeFactors(static_cast<std::size_t>(c)); // suspect, but it compiles
	…
}
```

### Scoped `enum`s may be forward-declared

```c++
enum Color; // error!
enum class Color; // fine
```

The inability to forward-declare `enum`s has drawbacks. The most notable is probably the increase in compilation dependencies:

Every `enum` in C++ has an integral underlying type that is determined by compilers. For an unscoped `enum` like `Color`, 

```c++
enum Color { black, white, red };
```

compilers might choose `char` as the underlying type, because there are only three values to represent. However, some enums have a range of values that is much larger:

```c++
enum Status { good = 0,
	failed = 1,
	incomplete = 100,
	corrupt = 200,
	indeterminate = 0xFFFFFFFF
};
```

Here the values to be represented range from `0` to `0xFFFFFFFF`. Compilers will have to select an integral type larger than `char` for the representation of `Status` values. 

To make efficient use of memory, compilers often want to choose the smallest underlying type for an `enum` that’s sufficient to represent its range of enumerator values. **To make that possible, C++98 supports only enum definitions (where all enumerators are listed); enum declarations are not allowed.**

The underlying type for a scoped `enum` is always known, and for unscoped `enum`s, you can specify it.

By default, the underlying type for scoped `enum`s is int:

```c++
enum class Status; // underlying type is int
```

If the default doesn’t suit you, you can override it:

```c++
enum class Status: std::uint32_t; 	// underlying type for
									// Status is std::uint32_t
									// (from <cstdint>)
```

To specify the underlying type for an unscoped `enum`, you do the same thing as for a scoped `enum`, and the result may be forward-declared:

```c++
enum Color: std::uint8_t; 	// fwd decl for unscoped enum;
							// underlying type is
							// std::uint8_t
```

### One situation where unscoped enums may be useful

That’s when referring to fields within C++11’s `std::tuples`. Suppose we have a tuple holding values for the name, email address, and reputation value for a user at a social networking website:

```c++
using UserInfo = // type alias; see Item 9
	std::tuple<std::string, // name
				std::string, // email
				std::size_t> ; // reputation
```

```c++
UserInfo uInfo; // object of tuple type
…
auto val = std::get<1>(uInfo); // get value of field 1, that is email address
```

Should you really be expected to remember that field 1 corresponds to the user’s email address? I think not. Using an unscoped enum to associate names with field numbers avoids the need to:

```c++
enum UserInfoFields { uiName, uiEmail, uiReputation };
UserInfo uInfo; // as before
…
auto val = std::get<uiEmail>(uInfo); // ah, get value of email field
```

What makes this work is the implicit conversion from `UserInfoFields` to `std::size_t`, which is the type that `std::get` requires.

The corresponding code with scoped `enum`s is substantially more verbose:

```c++
enum class UserInfoFields { uiName, uiEmail, uiReputation };
UserInfo uInfo; // as before
…
auto val = std::get<static_cast<std::size_t>(UserInfoFields::uiEmail)>(uInfo); // ah, get value of email field
```

## Item 11: Prefer `deleted` functions to `private` undefined ones
For the copy constructor and the copy assignment operator, the C++98 approach to preventing use of these functions is to declare them `private` and not define them. For example, To render `istream` and `ostream` classes uncopyable, `basic_ios` is specified in C++98 as follows:

```c++
template <class charT, class traits = char_traits<charT> >
class basic_ios : public ios_base {
public:
	…
private:
	basic_ios(const basic_ios& ); // not defined
	basic_ios& operator=(const basic_ios&); // not defined
};
```

Declaring these functions `private` prevents clients from calling them. Deliberately failing to define them means that if code that still has access to them (i.e., member functions or `friend`s of the class) uses them, linking will fail due to missing function definitions.

In C++11, there’s a better way to achieve essentially the same end: use “= `delete`” to declare them as as *deleted functions*.

```c++
template <class charT, class traits = char_traits<charT> >
class basic_ios : public ios_base {
public:
	…
	basic_ios(const basic_ios& ) = delete;
	basic_ios& operator=(const basic_ios&) = delete;
	…
};
```

Deleted functions may not be used in any way, so even code that’s in member and `friend` functions will fail to compile if it tries to copy `basic_ios` objects.

### `deleted` functions are declared `public`, not `private`

When client code tries to use a member function, C++ checks accessibility before deleted status. When client code tries to use a deleted `private` function, some compilers complain only about the function being `private`. Thus making the deleted functions `public` will generally result in **better error messages**.

### Any function may be deleted

Suppose we have a non-member function that takes an integer and returns whether it’s a lucky number:

```c++
bool isLucky(int number);
```

In C++, pretty much any type that can be viewed as vaguely numerical will implicitly convert to int, but some calls that would compile might not make sense:

```c++
if (isLucky('a')) … 	// is 'a' a lucky number?
if (isLucky(true)) … 	// is "true"?
if (isLucky(3.5)) … 	// should we truncate to 3
						// before checking for luckiness?
```

If lucky numbers must really be integers, we’d like to prevent calls such as these from compiling by **creating deleted overloads for the types we want to filter out**:

```c++
bool isLucky(int number); // original function
bool isLucky(char) = delete; // reject chars
bool isLucky(bool) = delete; // reject bools
bool isLucky(double) = delete; // reject doubles and floats
```

Although deleted functions can’t be used, they are part of your program. As such, they are taken into account during overload resolution:

```c++
if (isLucky('a')) … // error! call to deleted function
if (isLucky(true)) … // error!
if (isLucky(3.5f)) … // error!
```

### Prevent use of template instantiations that should be disabled

Suppose you need a template that works with built-in pointers:

```c++
template<typename T>
void processPointer(T* ptr);
```

In the case of the `processPointer` template, let’s assume the proper handling is to reject calls using `void*` pointers and `char*` pointers. That is, it should not be possible to call `processPointer` with `void*` or `char*` pointers.

```c++
template<>
void processPointer<void>(void*) = delete;

template<>
void processPointer<char>(char*) = delete;

template<>
void processPointer<const void>(const void*) = delete;

template<>
void processPointer<const char>(const char*) = delete;
```

If `processPointer` were a member function template inside `Widget`, for example, and you wanted to disable calls for `void*` pointers, this would be the C++98 approach, though it would not compile:

```c++
class Widget {
public:
	…
	template<typename T>
	void processPointer(T* ptr)
	{ … }
private:
	template<> // error!
	void processPointer<void>(void*);
};
```

The problem is that **template specializations must be written at namespace scope**, not class scope. This issue doesn’t arise for deleted functions, because they can be deleted outside the class (hence at namespace scope):

```c++
class Widget {
public:
	…
	template<typename T>
	void processPointer(T* ptr)
	{ … }
};

template<>
void Widget::processPointer<void>(void*) = delete;	// still public but deletede
```

## Item 12: Declare overriding functions `override`

Among the most fundamental ideas in OOP is that virtual function implementations in derived classes override the implementations of their base class counterparts. For overriding to occur, several requirements must be met:

* The base class function must be virtual

* The base and derived function names must be identical (except in the case of destructors)

* The parameter types of the base and derived functions must be identical

* The constness of the base and derived functions must be identical

* The return types and exception specifications of the base and derived functions must be **compatible**

* The functions’ *reference qualifiers* must be identical

All these requirements for overriding mean that small mistakes can make a big difference. For example, the following code is completely legal and, at first sight, looks reasonable, but it contains no virtual function overrides:

```c++
class Base {
public:
	virtual void mf1() const;
	virtual void mf2(int x);
	virtual void mf3() &;
	void mf4() const;
};
class Derived: public Base {
public:
	virtual void mf1();
	virtual void mf2(unsigned int x);
	virtual void mf3() &&;
	void mf4() const;
};
```

Because declaring derived class overrides is important to get right, but easy to get wrong, C++11 gives you a way to make explicit that a derived class function is supposed to override a base class version: *declare it `override`*.

```c++
class Derived: public Base {
public:
	virtual void mf1() override;
	virtual void mf2(unsigned int x) override;
	virtual void mf3() && override;
	virtual void mf4() const override;
};
```

This won’t compile, of course, because when written this way, compilers will kvetch about all the overriding-related problems.

### Contextual keywords

C++11 introduces two contextual keywords, `override` and `final` These keywords have the characteristic that they are reserved, but only in certain contexts. In the case of `override`, it has a reserved meaning only when it occurs at the end of a member function declaration:

```c++
class Warning { // potential legacy class from C++98
public:
	…
	void override(); // legal in both C++98 and C++11 (with the same meaning)
};
```

### Member function reference qualifiers
Reference qualifiers make it possible to limit use of a member function to lvalues only or to rvalues only:

  ```c++
  class Widget {
  public:
  	…
  	void doWork() &; 	// this version of doWork applies only when *this is an lvalue
  	void doWork() &&; 	// this version of doWork applies only when *this is an rvalue
  };
  ```
Reference qualifiers simply make it possible to draw the distinction for the object on which a member function is invoked, i.e., `*this`. It’s precisely
analogous to the `const` at the end of a member function declaration, which indicates that the object on which the member function is invoked (i.e., `*this`) is `const`.

Suppose our `Widget` class has a `std::vector` data member, and we offer an accessor function that gives clients direct access to it:

```c++
class Widget {
public:
    using DataType = std::vector<double>;
	DataType& data() { return values; }
	…
private:
	DataType values;
};
```

Consider what happens in this client code:

```c++
Widget w;
auto vals1 = w.data(); // copy w.values into vals1
```

The return type of `Widget::data` is an lvalue reference, and because lvalue references are defined to be lvalues, we’re initializing `vals1` from an lvalue. `vals1` is thus copy constructed from `w.values`.

```c++
// a factory function that creates Widgets
Widget makeWidget();
auto vals2 = makeWidget().data(); // copy values inside the Widget into vals2
```

Again, `Widgets::data` returns an lvalue reference, and, again, the lvalue reference is an lvalue, so, again, our new object (`vals2`) is copy constructed from values inside the `Widget`. This time, though, the `Widget` is the temporary object returned from `makeWidget` (i.e., an rvalue), so copying the `std::vector` inside it is a waste of time.

What’s needed is a way to specify that when `data` is invoked on an rvalue `Widget`, the result should also be an rvalue:

```c++
class Widget {
public:
	using DataType = std::vector<double>;
	DataType& data() & 					// for lvalue Widgets,
		{ return values; } 				// return lvalue
	DataType data() && 					// for rvalue Widgets,
		{ return std::move(values); } 	// return rvalue
	…
private:
	DataType values;
};
```

The lvalue reference overload returns an lvalue reference (i.e., an lvalue), and the rvalue reference overload returns a temporary object (i.e., an rvalue)

```c++
auto vals1 = w.data(); 					// calls lvalue overload for
										// Widget::data, copy-constructs vals1
auto vals2 = makeWidget().data(); 		// calls rvalue overload for
										// Widget::data, move-constructs vals2
```

