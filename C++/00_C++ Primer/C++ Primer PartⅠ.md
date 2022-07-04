# 1. Getting Started

## 1.1 A First Look at Input/Output

### 1.1.1 Standard Input and Output

* `cin`

  Standard input

* `cout`

  Standard output

* `cerr`

  Standard error

* `clog`

  For general information about the execution of the program

### 1.1.2 Writing to a Stream

```c++
std::cout << "Enter two numbers:" << std::endl;
```

1. The `<<` operator

   * Two operands:

     * left-hand operand:  an `ostream` object
     * right-hand operand: a value to print

   * Result

     <font color='red'>Its left operand. That is, the `ostream` on which we wrote the given value.</font>

2. `endl`: a special value called a *manipulator*

   * Effect

     * Ending the current line

     * <font color='red'>Flushing the buffer associated with that device</font>

       Ensures that all the output the program has generated so far is actually written to the output stream, rather than sitting in memory waiting to be written 

### 1.1.3 Using Names from the Standard Library

```c++
std::cout, std::endl
```

The scope operator `::` means that the names `cout` and `endl` are defined inside the namespace named `std`

### 1.1.4 Reading from a Stream

```c++
int v1 = 0, v2 = 0;
std::cin >> v1 >> v2;
```

The `>>` operator returns its left operand, in this case, the `std::cin`. 

## 1.2 Flow of Control

### 1.2.1 The `for` Statement

```c++
for (int val = 1; val <= 10; ++val)
    sum += val;
```

* init-statement: `int val = 1`
* condition: `val <= 10`
* expression: `++val`

The init-statement is only executed once, if the condition is valid, execute the body, and then the expression is executed, and the condition is retested...

### 1.2.2 ⭐Reading an Unknown Number of Inputs

```c++
int sum = 0, value = 0;
while(std::cin >> value){
    sum += value;
}
```

Because the `>>` operator returns its left operand, in this case, the `while` statement tests `std::cin`.

1. When we use a `istream` as a condition, the effect is to test the state of the stream:

   * If the stream is valid, then the test succeeds.

   * If the stream hits end-of-line or encouter an invalud input, it becomes invalid.

2. Entering an `End-of-File` from the Keyboard:

   * Windows system: Typing control-z, followed by hitting the Enter key

   * Unix system: control-d

## 1.3 Introducing Classes

1. <font color='red'>Every class defines a type</font>, the type name is the same as the name of the class.

   As with the built-in type, we can define a variable of a class type:

   ```c++
   Sales_item item; // Here the item is a Sales_item object
   ```

2. Member Function

   A member function is a function that is defined as part of a class

# 2. Variables and Basic Types

## 2.1 Primitive Built-in Types

### 2.1.1 Arithmetic Types

 ![image-20220630110110696](C++ Primer PartⅠ.assets/image-20220630110110696.png)

The arithmetic types are divided into two categories:

* integral types
  * character
  * bollean
* floating-point types

1. Signed and Unsigned Types

   The types `int`, `short`, `long`, and `long long` are all signed. We obtain the corresponding unsigned type by adding unsigned to the type, such as `unsigned long`. <font color='red'>The type `unsigned int` may be abbreviated as `unsigned`.</font>

2. Deciding which type to use
   * Use an unsigned type when you know that the value cannot be negative
   * Use `int` for integer arithmetic
   * `long` often has the same size as `int`, so when your data is larger than `int`, use `long long`
   * Use `double` for floating-point computations

### 2.1.2 Type Conversions

```c++
bool b = 42;			// The result is false if the value is 0 and true otherwise
int i = b;				// The result is 0 if the bool is false and 1 if the bool is true
unsigned char c = -1;	// Here the number of values unsigned char can hold is 256(0 to 255), so the result is
						// -1 % 256 = (-1 + 256) % 256 = 255
signed char c2 = 256;	// If we assign an out-of-range value to a signed type, the result is undefined
```

* When we assign one of the non`bool` arithmetic types to a `bool` object, the result is `false` if the value is `0` and `true` otherwise.
* When we assign a `bool` to one of the other arithmetic types, the resulting value is `1` if the `bool` is `true` and `0` if the `bool` is `false`.
* When we assign a floating-point value to an object of integral type, the value is truncated. The value that is stored is the part before the decimal point.
* When we assign an integral value to an object of floating-point type, the fractional part is zero. Precision may be lost if the integer has more bits than the floating-point object can accommodate.
* If we assign an out-of-range value to an object of unsigned type, <font color='red'>the result is the remainder of the value modulo the number of values the target type can hold</font>. For example, an 8-bit `unsigned char` can hold values from 0 through 255, inclusive. If we assign a value outside this range, the compiler assigns the remainder of that value modulo 256. Therefore, assigning –1 to an 8-bit `unsigned char` gives that object the value 255.
* <font color='red'>If we assign an out-of-range value to an object of signed type, the result is undefined.</font> The program might appear to work, it might crash, or it might
  produce garbage values.

#### ⭐Expressions Involving Unsigned Types

1. if we use both `unsigned` and `int` values in an arithmetic expression, the `int` value ordinarily is converted to `unsigned`.

   ```c++
   unsigned u = 10;
   int i = -42;
   std::cout << i + i << std::endl; // prints -84
   std::cout << u + i << std::endl; // if 32-bit ints, prints 4294967264
   ```

   In the second expression, the `int` value `-42` is converted to `unsigned` before the addition is done. Converting a negative number to `unsigned` makes the value “wraps around”.

2. Regardless of whether one or both operands are `unsigned`, if we subtract a value from an `unsigned`, we must be sure that the result cannot be negative:

   ```c++
   unsigned u1 = 42, u2 = 10;
   std::cout << u1 - u2 << std::endl; // ok: result is 32
   std::cout << u2 - u1 << std::endl; // ok: but the result will wrap around
   ```

3. <font color='red'>The fact that an `unsigned` cannot be less than zero also affects how we write loops.</font>

   ```c++
   for(unsigned u = 10,; u >=0; --u)
       std::cout << u;
   ```

   When `u` is `0`, we print the value `0`, and `--u` subtracts `1` from `u`. That result, `-1`, will not fit in an `unsigned` type. `-1` will be converted to `unsigned`, resulting `4294967295`, so the loop will continue.

   One way to write this loop is to use a `while` instead of a for. Using a `while` lets us decrement before (rather than after) printing our value:

   ```c++
   unsigned u = 11;	// start the loop one past the first element we want to print
   while(u >= 1){
       --u;
       std::cout << u;	// decrement first, so that the last iteration will print 0
   }
   ```

<font color='orange'>Caution: Don’t Mix `Signed` and `Unsigned` Types</font>

### 2.1.3 Literals

A value, such as `42`, is known as a literal because its <font color='red'>value self-evident</font>. Every literal has a type. The form and value of a literal determine its type.

#### Integer and Floating-Point Literals

1. We can write an integer literal using decimal, octal, or hexadecimal notation. 

   Integer literals that begin with `0` (zero) are interpreted as octal. Those that begin with either `0x` or `0X` are interpreted as hexadecimal.

   ```c++
   20 /* decimal */ 024 /* octal */ 0x14 /* hexadecimal */
   ```

2. The type of an integer literal depends on its value and notation.

   By default, floating-point literals have type double. We can override defaults by using a suffix.

    ![image-20220630115246277](C++ Primer PartⅠ.assets/image-20220630115246277.png)

   ```c++
   L'a' // wide character literal, type is wchar_t
   u8"hi!" // utf-8 string literal (utf-8 encodes a Unicode character in 8 bits)
   42ULL // unsigned integer literal, type is unsigned long long
   1E-3F // single-precision floating-point literal, type is float
   3.14159L // extended-precision floating-point literal, type is long double
   ```

3. <font color='red'>The value of a decimal literal is never a negative number.</font>

   If we write `-42`, the minus sign is not part of the literal. The minus sign is an operator that negates the value of its operand.

#### Character and Character String Literals

A character enclosed within single quotes is a literal of type `char`. Zero or more characters enclosed in double quotation marks is a *string literal*.

<font color='red'>Two string literals that appear adjacent to one another and that are separated only by spaces, tabs, or newlines are concatenated into a single literal.</font>

```c++
// multiline string literal
std::cout << "a really, really long string literal "
                     "that spans two lines" << std::endl;
```

#### Boolean and Pointer Literals

The words `true` and `false` are literals of type `bool`:

```c++
bool test = false;
```

The word `nullptr` is a pointer literal.

## 2.2. Variables

### 2.2.1. Variable Definitions

Simple variable definition consists of a *type specifier*, followed by a list of one or more variable names separated by commas, and ends with a semicolon.

<font color='red'>An Object in C++:
An object is a region of memory that contain data and has a type.</font>

#### Initializers

When a definition defines two or more variables, the name of each object becomes visible immediately. Thus, it is possible to initialize a variable to the value of one defined earlier in the same definition.

```c++
// ok: price is defined and initialized before it is used to initialize discount
double price = 109.99, discount = price * 0.16;
```

#### List Initialization

We can use any of the following four different ways to define an `int` variable named `units_sold` and initialize it to `0`:

```c++
int units_sold = 0;
int units_sold = {0};
int units_sold{0};
int units_sold(0);
```

Braced lists of initializers can be used whenever we initialize an object and in some cases when we assign a new value to an object.

When used with variables of built-in type, list initialization has one important property: <font color='red'>The compiler will not let us list initialize variables of built-in type if
the initializer might lead to the loss of information</font>:

```c++
long double ld = 3.1415926536;
int a{ld}, b = {ld}; // error: narrowing conversion required
int c(ld), d = ld; // ok: but value will be truncated
```

#### Default Initialization

When we define a variable without an initializer, the variable is <font color='cornflowerblue'>**default initialized**</font>. Such variables are given the “default” value:

* built-in type

  * Variables defined outside any function body are initialized to zero
  * <font color='red'>Variables defined inside a function are uninitialized</font>. The value of an uninitialized variable of built-in type is undefined.

* class type

  Each class controls how we initialize objects of that class type. In particular, it is up to the class whether we can define objects of that type without an initializer. If we can, the class determines what value the resulting object will have.

  For example, as we’ve just seen, the library `string` class says that if we do not supply an initializer, then the resulting string is the empty string:

  ```c++
  std::string empty; // empty implicitly initialized to the empty string
  Sales_item item; // default-initialized Sales_item object
  ```

  Some classes require that every object be explicitly initialized. The compiler will complain if we try to create an object of such a class with no initializer.

### 2.2.2. Variable Declarations and Definitions

A **variable declaration** specifies the type and name of a variable. 

A **variable definition** is a declaration. In addition to specifying the name and type, a definition also allocates storage and may provide the variable with an initial value.

Variables must be **defined** exactly once but can be **declared** many times.

1. `extern` keyword

   To obtain a declaration that is not also a definition, we add the `extern` keyword and may not provide an explicit initializer:

   ```c++
   extern int i; // declares but does not define i
   int j; // declares and defines j
   ```

   Any declaration that includes an explicit initializer is a definition. We can provide an initializer on a variable defined as `extern`, but doing so overrides the `extern`. An `extern` that has an initializer is a definition:

   ```c++
   extern double pi = 3.1416; // definition
   ```

2. Static Typing

   C++ is a statically typed language, which means that **types are checked at compile time**. 

   As our programs get more complicated, we’ll see that static type checking can help find bugs. However, a consequence of static checking is that the
   type of every entity we use must be known to the compiler. As one example, we must declare the type of a variable before we can use that variable.

### 2.2.3. Identifiers 标识符

Identifiers in C++ can be composed of **letters**, **digits**, and the **underscore character**.

<font color='red'>Identifiers must begin with either a letter or an underscore.</font>

#### Conventions for Variable Names

1. An identifier should give some indication of its meaning.
2. Variable names normally are lowercase—`index`, not `Index` or `INDEX`.
3. Like `Sales_item`, classes we define usually begin with an uppercase letter.
4. Identifiers with multiple words should visually distinguish each word, for example, `student_loan` or `studentLoan`, not `studentloan`.

### 2.2.4. Scope of a Name

1. Advice: Define Variables Where You First Use Them

2. Use a scope operator to requests the global variable:

   ```c++
   #include <iostream>
   
   int reused = 42;
   int main(){
       int reused = 0;
       std::cout << reused << std::endl;
       std::cout << ::reused << std::endl;
   }
   ```

   The global scope has no name. Hence, when the scope operator has an empty left-hand side, it is a request to fetch the name on the right-hand side from the global scope

## 2.3 Compound Types 复合类型

A compound type is a type that is defined in terms of another type

### 2.3.1. References

* A reference defines an alternative name for an object

* Define a reference type by writing a declarator of the form `&d`. where `d` is the name being declared.

  ```c++
  int ival = 1024;
  int &refVal = ival;    //refVal is another name for ival
  ```

* After a reference has been defined, all operations on that reference are actually operations on the object which the reference is bound.

* A reference must be initialized, and there is no way to rebind a reference to refer to a different object.

* A reference is not an object. Instead a reference is just another name for an already existing object.

* The type of a reference and the object to which the reference refers must match exactly. (With two exceptions)

  ```c++
  double dval =3.14;
  int &refval1 = dval;    //error
  ```

* A reference may be bound only to an object, not to a literal or to another reference

  ```c++
  int &refVal = 10;    //error
  ```

### 2.3.2. Pointers

<font color='red'>Difference with reference</font>

* A pointer is an object, it can be assigned and copied

* A single pointer can point to several different objects over its lifetime

* A pointer need not be initialized at the time it is defined.

   Like other built-in types, pointers defined at block scope have undefined value if they are not initialized

We define a pointer type by writing a declarator of the form `*d`, where `d` is the name being defined. <font color='red'>The `*` must be repeated for each pointer variable</font>:

```c++
int *ip1, *ip2; // both ip1 and ip2 are pointers to int
double dp, *dp2; // dp2 is a pointer to double; dp is a double
```

#### Taking the Address of an Object

A pointer holds the address of another object. We get the address of an object by using the address-of operator (the `&` operator):

```c++
int ival = 42;
int *p = &ival;
```

<font color='red'>Because references are not objects, they don’t have addresses. Hence, we may not define a pointer to a reference.</font>

With two exceptions, the types of the pointer and the object to which it points must match:

```c++
double dval;
double *pd = &dval; // ok: initializer is the address of a double
double *pd2 = pd; // ok: initializer is a pointer to double
int *pi = pd; // error: types of pi and pd differ
pi = &dval; // error: assigning the address of a double to a pointer to int
```

#### Using a Pointer to Access an Object

When a pointer points to an object, we can use the dereference operator (the `*` operator) to access that object:

```c++
int ival = 42;
int *p = &ival; 	// p holds the address of ival; p is a pointer to ival
cout << *p; 		// * yields the object to which p points; prints 42
*p = 0; 			// * yields the object; we assign a new value to ival through p
cout << *p; 		// prints 0
```

#### Null Pointers

There are several ways to obtain a null pointer:

```c++
int *p1 = nullptr; 		// equivalent to int *p1 = 0;
int *p2 = 0; 			// directly initializes p2 from the literal constant 0
// must #include cstdlib
int *p3 = NULL; 		// equivalent to int *p3 = 0;
```

It is illegal to assign an `int` variable to a pointer, even if the variable’s value happens to be `0`.

```c++
int zero = 0;
pi = zero; // error: cannot assign an int to a pointer
```

#### Assignment and Pointers

Assignment makes the pointer point to a different object:

```c++
int ival = 42;
int *pi = 0; 	// pi is initialized but addresses no object
pi = &ival; 	// value in pi is changed; pi now points to ival
*pi = 0; 		// value in ival is changed; pi is unchanged
```

#### Other Pointer Operations

1. We can use a pointer in a condition. If the pointer is 0, then the condition is `false`. Any nonzero pointer evaluates as `true`.
2. Given two valid pointers of the same type, we can compare them using the equality (`==`) or inequality (`!=`) operators. 

#### `void*` Pointers

The type `void*` is a special pointer type that can hold the address of any object.

```c++
double obj = 3.14, *pd = &obj;
void *pv = &obj;
pv = pd;
```

There are only a limited number of things we can do with a `void*` pointer:

* Compare it to another pointer
* Pass it to or return it from a function
* Assign it to another void* pointer

<font color='red'>We cannot use a `void*` to operate on the object it addresses</font>—we don’t know that object’s type, and the type determines what operations we can perform on the object.

### 2.3.3. Understanding Compound Type Declarations

#### Defining Multiple Variables

```c++
int* p; // legal but might be misleading
```

We say that this definition might be misleading because it suggests that `int*` is the type of each variable declared in that statement. Despite appearances, <font color='red'>the base type of this declaration is `int`, not `int*`. The `*` modifies the type of `p`.</font>

```c++
int* p1, p2; // p1 is a pointer to int; p2 is an int
int *p1, *p2; // both p1 and p2 are pointers to int
```

#### Pointers to Pointers

We indicate each pointer level by its own `*`. That is, we write `**` for a pointer to a pointer, `***` for a pointer to a pointer to a pointer, and so on:

```c++
int ival = 1024;
int *pi = &ival; // pi points to an int
int **ppi = &pi; // ppi points to a pointer to an int

cout << "The value of ival\n"
	<< "direct value: " << ival << "\n"
	<< "indirect value: " << *pi << "\n"
	<< "doubl
```

#### ⭐References to Pointers

```c++
int i = 42;
int *p;
int *&r = p;
r = &i;
*r = 0;
```

The easiest way to understand the type of `r` is to read the definition right to left:

* The symbol closest to the name of the variable (in this case the `&` in `&r`) is the one that has the most immediate effect on the variable’s type. Thus, we know that `r` is a reference.
* The rest of the declarator determines the type to which `r` refers. The next symbol, `*` in this case, says that the type `r` refers to is a pointer type.
* Finally, the base type of the declaration says that `r` is a reference to a pointer to an `int`.

## 2.4. `const` Qualifier

Because we can’t change the value of a `const` object after we create it, it **must be initialized**. As usual, the initializer may be an arbitrarily complicated expression:

```c++
const int i = get_size(); 	// ok: initialized at run time
const int j = 42; 			// ok: initialized at compile time
const int k; 				// error: k is uninitialized const
```

#### By Default, `const` Objects Are Local to a File

When a `const` object is initialized from a compile-time constant, the compiler will usually replace uses of the variable with its corresponding value
during compilation. 

To substitute the value for the variable, the compiler has to see the variable’s initializer. When we split a program into multiple files, every file that uses the `const` must have access to its initializer. In order to see the initializer, <font color='red'>the variable must be defined in every file that wants to use the variable’s value</font>. 

To support this usage, yet avoid multiple definitions of the same variable, `const` variables are defined as local to the file. <font color='red'>When we define a const with the same name in multiple files, it is as if we had written definitions for separate variables in each file.</font>

### 2.4.1. References to `const`

Unlike an ordinary reference, a reference to `const` cannot be used to change the object to which the reference is bound:

```c++
const int ci = 1024;
const int &r1 = ci;		// ok: both reference and underlying object are const
r1 = 42;				// error: r1 is a reference to const
int &r2 = ci;			// error: non const reference to a const object
```

>Terminology: `const` Reference is a Reference to `const`
>
>Technically speaking, there are no `const` references. A reference is not an object, so we cannot make a reference itself `const`.

#### ⭐Initialization and References to `const`

<font color='red'>We can initialize a reference to `const` from any expression that can be converted to the type of the reference. In particular, we can bind a reference to
`const` to a non`const` object, a literal, or a more general expression:</font>

```c++
int i = 42;
const int &r1 = i;			// we can bind a const int& to a plain int object
const int &r2 = 42;			// ok: r1 is a reference to const
const int &r3 = r1 * 2;		// ok: r3 is a reference to const
int &r4 = r * 2;			// error: r4 is a plain, non const reference
```

Consider what happens when we bind a reference to an object of a different type:

```c++
double dval = 3.14;
const int &r1 = dval;
```

To ensure that the object to which `r1` is bound is an `int`, the compiler transforms this code into something like

```c++
const int temp = dval;	// create a temporary const int from the double
const int &ri = temp;	// bind ri to that temporary
```

In this case, `ri` is bound to a temporary object. A temporary object is an unnamed object created by the compiler when it needs a place to store a result from evaluating an expression.

#### A Reference to `const` May Refer to an Object That Is Not `const`

It is important to realize that a reference to `const` restricts only what we can do through that reference. Binding a reference to `const` to an object says nothing about whether the underlying object itself is `const`. Because the underlying object might be non`const`, it might be changed by other means:

```c++
int i = 42;
int &r1 = i; 			// r1 bound to i
const int &r2 = i; 		// r2 also bound to i; but cannot be used to change i
r1 = 0; 				// r1 is not const; i is now 0
r2 = 0; 				// error: r2 is a reference to const
```

### 2.4.2. Pointers and const

Like a reference to `const`, a pointer to `const` may not be used to change the object to which the pointer points. We may store the address of a `const` object only in a pointer to `const`:

```c++
const double pi = 3.14;
double *ptr = &pi;			// error: ptr is a plain pointer
const double *cptr = &pi;	// ok: cptr may point to a double that is const
*cptr = 42;					// error: cannot assign to *cptr
```

<font color='red'>We can use a pointer to const to point to a nonconst object</font>

```c++
double dval = 3.14;
const double *cptr = &dval;
```

#### `const` Pointers

Unlike references, pointers are objects. Hence, as with any other object type, we can have a pointer that is itself `const`.

<font color='red'>We indicate that the pointer is `const` by putting the `const` after the `*`.</font> This placement indicates that it is the pointer, not the pointed-to type, that is
`const`:

```c++
int errNumb = 0;
int *const curErr = &errNumb;		// curErr will always point to errNumb
const double pi = 3.14;
const double *const pip = &pi;		// pip is a const pointer to a const object
```

The easiest way to understand these declarations is to <font color='red'>read them from right to left</font>. 

* In this case, the symbol closest to `curErr` is `const`, which means that `curErr` itself will be a `const` object. 
* The type of that object is formed from the rest of the declarator. The next symbol in the declarator is `*`, which means that `curErr` is a const pointer.
* Finally, the base type of the declaration completes the type of `curErr`, which is a `const` pointer to an object of type `int`.

### 2.4.3. Top-Level const

We use the term<font color='blue'> top-level `const`</font> to indicate that the pointer itself is a `const`. When a pointer can point to a `const` object, we refer to that `const` as a <font color='blue'>low-level const</font>.

More generally, top-level `const` indicates that an object itself is `const`. Top-level `const` can appear in any object type, i.e., one of the built-in arithmetic types, a class type, or a pointer type. 

Low-level const appears in the base type of compound types such as pointers or references. 

```c++
int i = 0;
int *const p1 = &i; 		// we can't change the value of p1; const is top-level
const int ci = 42; 			// we cannot change ci; const is top-level
const int *p2 = &ci; 		// we can change p2; const is low-level
const int *const p3 = p2; 	// right-most const is top-level, left-most is not
const int &r = ci; 			// const in reference types is always low-level
```

1. <font color='red'>When we copy an object, top-level `const`s are ignored</font>

   ```c++
   i = ci; 		// ok: copying the value of ci; top-level const in ci is ignored
   p2 = p3; 		// ok: pointed-to type matches; top-level const in p3 is ignored
   ```

2. On the other hand, low-level `const` is never ignored. When we copy an object, both objects must have the same low-level `const` qualification or there must be a conversion between the types of the two objects. In general, we can convert a non`const` to `const` but not the other way round:

   ```c++
   int *p = p3;		// error: p3 has a low-level const but p doesn't
   p2 = p3;			// ok: p2 has the same low-level const qualification as p3
   p2 = &i;			// ok: we can convert int* to const int*
   int &r = ci;		// error: can't bind an ordinary int& to a const int object
   const int &r = i;	// ok: can bind const int& to plain int
   ```

### 2.4.4. `constexpr` and Constant Expressions

#### Constant Expression

A constant expression is an expression whose value cannot change and that <font color='red'>can be evaluated at compile time</font>:

* A literal is a constant expression.
* A const object that is initialized from a constant expression is also a constant expression.

```c++
const int max_files = 20; 			// max_files is a constant expression
const int limit = max_files + 1; 	// limit is a constant expression
int staff_size = 27; 				// staff_size is not a constant expression
const int sz = get_size(); 			// sz is not a constant expression
```

Although `staff_size` is initialized from a literal, it is not a constant expression because it is a plain `int`, not a `const int`. On the other hand, even though `sz` is a `const`, the value of its initializer is not known until run time. Hence, `sz` is not a constant expression.

#### `constexpr` Variables

In a large system, it can be difficult to determine (for certain) that an initializer is a constant expression. Under the new standard, we can ask the compiler to verify that a variable is a constant expression by declaring the variable in a `constexpr` declaration. 

<font color='red'>Variables declared as `constexpr` are implicitly `const` and must be initialized by constant expressions:</font>

```c++
constexpr int mf = 20;			// 20 is a constant expression
constexpr int limit = mf + 1;	// mf + 1 is a constant expression
constexpr int sz = size();		// ok only if size is a constexpr function
```

Although we cannot use an ordinary function as an initializer for a `constexpr` variable, <font color='red'>we can use `constexpr` functions in the initializer of a `constexpr` variable.</font> Such functions must be simple enough that the compiler can evaluate them at compile time.

#### Literal Types

Because a constant expression is one that can be evaluated at compile time, there are limits on the types that we can use in a `constexpr` declaration. The types we can use in a `constexpr` are known as “<font color='blue'>literal types</font>” because they are simple enough to have literal values:

* the arithmetic types
* reference types
* pointer  types

Our `Sales_item` class and the library IO and `string` types are not literal types. Hence, we cannot define variables of these types as `constexpr`s.

1. Although we can define both pointers and reference as `constexpr`s, the objects we use to initialize them are strictly limited:
   * We can initialize a `constexpr` pointer from the `nullptr` literal or the literal `0`. 
   * We can also point to (or bind to) an object that remains at a fixed address
     * Variables defined inside a function ordinarily are not stored at a fixed address. Hence, we cannot use a `constexpr` pointer to point to such variables.
     * The address of an object defined outside of any function is a constant expression, and so may be used to initialize a `constexpr` pointer.
     * Functions may define variables that exist across calls to that function. Like an object defined outside any function, these special local objects also have fixed addresses. Therefore, a `constexpr` reference may be bound to, and a `constexpr` pointer may address, such variables.

#### Pointers and `constexpr`

It is important to understand that when we define a pointer in a `constexpr` declaration, <font color='red'>the `constexpr` specifier applies to the pointer, not the type to which the pointer points</font>:

```c++
const int *p = nullptr;			// p is a pointer to a const int
constexpr int *np = nullptr;	// np is a constant pointer to int that is null
int j = 0;
constexpr int i = 42;
// i and j must be defined outside any function
constexpr const int *p = &i;	// p is a constant pointer to the const int i
constexpr int *p1 = &jl			// p1 is a constant pointer to the int j
```

> `constexpr` imposes a top-level `const` on the objects it defines.

## 2.5. Dealing with Types

### 2.5.1. Type Aliases

We can define a type alias in one of two ways:

* `typedef`

  <font color='red'>The keyword `typedef` may appear as part of the base type of a declaration.</font> Declarations that include typedef define type aliases rather than variables. As in any other declaration, the declarators can include type modifiers that define compound types built from the base type of the definition:

  ```c++
  typedef double wages;		// wage is a synonym for double
  typedef wages base, *p;		// base is a synonym for double, p is a synonym for double *
  ```

* alias declaration

  The alias declaration defines the name on the left-hand side of the `=` as an alias for the type that appears on the right-hand side.

  ```c++
  using SI = Sales_item; 		// SI is a synonym for Sales_item
  ```

#### Pointers, const, and Type Aliases

```c++
typedef char *pstring;			// pstring is a synonym for char*
const pstring cstr = 0;			// cstr is a constant pointer to char
const pstring *ps;				// ps is a pointer to a constant pointer to char
```

The base type in these declarations is `const pstring`. As usual, a `const` that appears in the base type modifies the given type. The type of `pstring` is “pointer to `char`.” So, `const pstring` is a constant pointer to `char`—not a pointer to `const char`.

### 2.5.2. The `auto` Type Specifier

Under the new standard, we can let the compiler figure out the type for us by using the `auto` type specifier. `auto` tells the compiler to deduce the type
from the initializer. By implication, a variable that uses `auto` as its type specifier must have an initializer:

```c++
// the type of item is deduced from the type of the result of adding val1 and val2
auto item = val1 + val2; // item initialized to the result of val1 + val2
```

As with any other type specifier, we can define multiple variables using `auto`. Because a declaration can involve only a single base type, the initializers for all the variables in the declaration must have types that are consistent with each other:

```c++
auto i = 0, *p = &i;		// ok: i is int and p is a pointer to int
auto sz = 0, pi = 3.14;		// error: inconsistent types for sz and pi
```

#### Compound Types, `const`, and `auto`

The type that the compiler infers for `auto` is not always exactly the same as the initializer’s type. 

Instead, the compiler adjusts the type to conform to normal initialization rules:

* When we use a reference, we are really using the object to which the reference refers. In particular, when we use a reference as an initializer, the
  initializer is the corresponding object. The compiler uses that object’s type for `auto`’s type deduction:

  ```c++
  int i = 0, &r = i;
  auto a = r;			// a is an int
  ```

* `auto` ordinarily ignores top-level `const`s. As usual in initializations, low-level `const`s, such as when an initializer is a pointer to `const`, are
  kept:

  ```c++
  const int ci = i, &cr = ci;
  auto b = ci; 	// b is an int (top-level const in ci is dropped)
  auto c = cr; 	// c is an int (cr is an alias for ci whose const is top-level)
  auto d = &i; 	// d is an int*(& of an int object is int*)
  auto e = &ci; 	// e is const int*(& of a const object is low-level const)
  ```

  If we want the deduced type to have a top-level `const`, we must say so explicitly:

  ```c++
  const auto f = ci;	// deduced type of ci is int; f has type const int
  ```

* When we ask for a reference to an auto-deduced type, top-level `const`s in the initializer are not ignored. As usual, `const`s are not top-level when we bind a reference to an initializer.

  ```c++
  auto &g = ci;		// g is a const int& that is bound to ci
  auto &h = 42;		// error: we can't bind a plain reference to a literal
  const auto &j = 42;	// ok: we can bind a const reference to a literal
  ```

> auto 三条规则：
>
> * 忽略引用
> * 声明的是指针的时候，忽略顶层const
> * 声明的是引用的时候，保留顶层const

When we define several variables in the same statement, it is important to remember that a reference or pointer is part of a particular declarator and not part of the base type for the declaration. As usual, the initializers must provide consistent `auto`-deduced types:

```c++
auto k = ci, &l = i;	// k is int; l is int&
auto &m = ci, *p = &ci;	// m is a const int&, p is a pointer to const int
auto &n = i, *p2 = &ci;	// error: type deduced from i is int; type deduced from &ci is const int
```

### 2.5.3. The `decltype` Type Specifier

Sometimes we want to define a variable with a type that the compiler deduces from an expression but do not want to use that expression to initialize the variable. For such cases, the new standard introduced a second type specifier, `decltype`, which returns the type of its operand.

```c++
decltype(f()) sum = x; // sum has whatever type f returns
```

When the expression to which we apply `decltype` is a variable, <font color='red'>`decltype` returns the type of that variable, including top-level const and references</font>:

```c++
const int ci = 0, &cj = ci;
decltype(ci) x = 0;		// x is const int
decltype(cj) y = x;		// y is const int&
decltype(cj) z;			// error: z is a reference and must be initialized
```

#### `decltype` and References

When we apply `decltype` to an expression that is not a variable, we get the type that that expression yields. <font color='red'>`decltype` returns a reference type for expressions that yield objects that can stand on the left-hand side of the assignment</font>:

```c++
// decltype of an expression can be a reference type
int i = 42, *p = &i, &r = i;
decltype(r + 0) b;	// ok: addition yields an int; b is an (uninitialized) int
decltype(*p) c;		// error: c is int& and must be initialized
```

* Here `r` is a reference, so `decltype(r)` is a reference type. If we want the type to which `r` refers, we can use `r` in an expression, such as `r + 0`, which is an expression that yields a value that has a nonreference type.
* The dereference operator is an example of an expression for which `decltype` returns a reference. As we’ve seen, when we dereference a pointer,
  we get the object to which the pointer points. Moreover, we can assign to that object. Thus, the type deduced by `decltype(*p)` is `int&`, not plain `int`.

Enclosing the name of a variable in parentheses affects the type returned by `decltype`. When we apply `decltype` to a variable without any parentheses, we get the type of that variable. <font color='red'>If we wrap the variable’s name in one or more sets of parentheses, the compiler will evaluate the operand as an expression</font>:

```c++
// decltype of a parenthesized variable is always a reference
decltype((i)) d; // error: d is int& and must be initialized
decltype(i) e; // ok: e is an (uninitialized) int
```

## 2.6. Defining Our Own Data Structures

## 2.6.1. Defining the `Sales_data` Type

```c++
struct Sales_data{
    std::string bookNo;
    unsigned units_sold = 0;
    double revenue = 0.0;
};
```

The close curly that ends the class body must be followed by a semicolon. The semicolon is needed because we can define variables after the class body:

```c++
struct Sales_data { /* ... */ } accum, trans, *salesptr;
// equivalent, but better way to define these objects
struct Sales_data { /* ... */ };
Sales_data accum, trans, *salesptr;
```

#### Class Data Members

Each object has its own copy of the class data members. Modifying the data members of one object does not change the data in any other `Sales_data`
object.

Under the new standard, we can supply an <font color='blue'>in-class initializer</font> for a data member. When we create objects, the in-class initializers will be used to initialize the data members. <font color='red'>Members without an initializer are default initialized.</font> 

In-class initializers are restricted as to the form we can use: They must either be enclosed inside curly braces or follow an `=` sign. <font color='red'>We may not specify an
in-class initializer inside parentheses.</font>

### 2.6.2. Using the `Sales_data` Class

```c++
#include <iostream>
#include <string>
#include "Sales_data.h"
int main()
{
	Sales_data data1, data2;
	// code to read into data1 and data2
	double price = 0;
    std::cin >> data1.bookNo >> data1.units_sold >> price;
    data1.revenue = data1.units_sold * price;
    std::cin >> data2.bookNo >> data2.units_sold >> price;
	data2.revenue = data2.units_sold * price;
	// code to check whether data1 and data2 have the same ISBN
	// and if so print the sum of data1 and data2
    if (data1.bookNo == data2.bookNo) {
		unsigned totalCnt = data1.units_sold + data2.units_sold;
		double totalRevenue = data1.revenue + data2.revenue;
        std::cout << data1.bookNo << " " << totalCnt
					<< " " << totalRevenue << " ";
	}
}
```

### 2.6.3. Writing Our Own Header Files

In order to ensure that the class definition is the same in each file, classes are usually defined in header files. Headers (usually) contain entities (such as class definitions and `const` and `constexpr` variables) that can be defined only once in any given file.

However, headers often need to use facilities from other headers. For example, because our `Sales_data` class has a `string` member, `Sales_data`.h must `#include` the `string` header. As we’ve seen, programs that use `Sales_data` also need to include the `string` header in order to use the `bookNo` member. As a result, programs that use `Sales_data` will include the `string` header twice.

Because a header might be included more than once, we need to write our headers in a way that is safe even if the header is included multiple times.

#### A Brief Introduction to the Preprocessor

The preprocessor—which C++ inherits from C—is a program that runs before the compiler and changes the source text of our programs. C++ programs also use the preprocessor to define <font color='blue'>header guards</font>.

Header guards rely on preprocessor variables. Preprocessor variables have one of two possible states: defined or not defined.

* The `#define` directive takes a name and defines that name as a preprocessor variable.
* `#ifdef` is true if the variable has been defined
* `#ifndef` is true if the variable has not been defined

```c++
#ifndef SALES_DATA_H
#define SALES_DATA_H
#include <string>
struct Sales_dat{
    std::string bookNo;
	unsigned units_sold = 0;
	double revenue = 0.0;
};
#endif
```

The first time `Sales_data.h` is included, the `#ifndef` test will succeed. The preprocessor will process the lines following `#ifndef` up to the `#endif`. As a result, the preprocessor variable `SALES_DATA_H` will be defined and the contents of `Sales_data.h` will be copied into our program. 

If we include `Sales_data.h` later on in the same file, the `#ifndef` directive will be false. The lines between it and the `#endif` directive will be ignored.