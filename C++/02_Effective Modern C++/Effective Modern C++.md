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