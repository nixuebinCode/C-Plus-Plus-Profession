# CHAPTER 1 Deducing Types
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

