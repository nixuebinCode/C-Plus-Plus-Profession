# Chapter 17. Specialized Library Facilities

## 17.1. The `tuple` Type

A `tuple` is a template that is similar to a `pair`.  Each `pair` type has different types for its members, but every pair always has exactly two members. A `tuple` also has members whose types vary from one `tuple` type to another, but a `tuple` can have any number of members. Each distinct `tuple` type has a fixed number of members, but the number of members in one `tuple` type can differ from the number of members in another.

A `tuple` is most useful when we want to combine some data into a single object but do not want to bother to define a data structure to represent those data.

 ![image-20220829143738795](images/image-20220829143738795.png)

### 17.1.1. Defining and Initializing `tuple`s

When we define a `tuple`, we name the type(s) of each of its members:

```c++
tuple<size_t, size_t, size_t> threeD;		// all three members set to 0
tuple<string, vector<double>, int, list<int>> someVal("consttants", {3.14, 2.718}, 
                                                     	42, {0, 1, 2, 3, 4, 5});
```

When we create a `tuple` object, we can use the default tuple constructor, which value initializes each member, or we can supply an initializer for each member as we do in the initialization of `someVal`. This `tuple` constructor is `explicit`, so we must use the direct initialization syntax.

Alternatively, similar to the `make_pair` function, the library defines a `make_tuple` function that generates a `tuple` object:

```c++
auto item = make_tuple("0-999-78345-X", 3, 20.00);
```

Like `make_pair`, the `make_tuple` function uses the types of the supplied initializers to infer the type of the `tuple`. In this case, `item` is a `tuple` whose type is `tuple<const char*, int, double>`.

#### Accessing the Members of a tuple

We access the members of a `tuple` through a library function template named `get`. 

To use `get` we must specify an explicit template argument, which is the position of the member we want to access. We pass a `tuple` object to `get`, which returns a reference to the specified member:

```c++
auto book = get<0>(item);			// returns the first member of item
auto cnt = get<1>(item); 			// returns the second member of item
auto price = get<2>(item)/cnt; 		// returns the last member of item
```

#### Relational and Equality Operators

The `tuple` relational and equality operators behave similarly to the corresponding operations on containers. These operators execute pairwise on the members of the left-hand and right-hand `tuple`s.

We can compare two `tuple`s only if they have the same number of members.

Moreover, to use the equality or inequality operators, it must be legal to compare each pair of members using the `==` operator;
to use the relational operators, it must be legal to use `<`.

```c++
tuple<string, string> duo("1", "2");
tuple<size_t, size_t> twoD(1, 2);
bool b = (duo == twoD); 	// error: can't compare a size_t and a string
tuple<size_t, size_t, size_t> threeD(1, 2, 3);
b = (twoD < threeD); 		// error: differing number of members
tuple<size_t, size_t> origin(0, 0);
b = (origin < twoD); 		// ok: b is true
```

### 17.1.2. Using a `tuple` to Return Multiple Values

**<font color='red'>A common use of `tuple` is to return multiple values from a function.</font>**

## 17.2. The `bitset` Type

The standard library defines the `bitset` class to make it easier to use bit operations and possible to deal with collections of bits that are larger than the longest integral type.

### 17.2.1. Defining and Initializing `bitset`s

The bitset class is a class template that, like the `array` class, has a fixed size. **<font color='red'>When we define a `bitset`, we must say how many bits the `bitset` will contain:</font>**

```c++
bitset<32> bitvec(1U);	// 32 bits; low-order bit is 1, remaining bits are 0
```

#### Initializing a `bitset` from an unsigned Value

When we use an integral value as an initializer for a `bitset`, that value is converted to `unsigned long long` and is treated as a bit pattern. The bits in the `bitset` are a copy of that pattern.

#### Initializing a `bitset` from a `string`

We can initialize a `bitset` from either a `string` or a pointer to an element in a character array. In either case, **<font color='red'>the characters represent the bit pattern directly</font>**:

```c++
bitset<32> bitvec4("1100"); // bits 2 and 3 are 1, all others are 0
```

We need not use the entire `string` as the initial value for the `bitset`. Instead, we can use a substring as the initializer:

```c++
string str("1111111000000011001101");
bitset<32> bitvec5(str, 5, 4); 			// four bits starting at str[5], 1100
bitset<32> bitvec6(str, str.size()-4); 	// use last four characters
```

### 17.2.2. Operations on `bitset`s

 ![image-20220829151020862](images/image-20220829151020862.png)

The `bitset` class also supports the bitwise operators that we covered in § 4.8. The operators have the same meaning when applied to `bitset` objects as the built-in operators have when applied to unsigned operands.

#### `bitset` IO Operators

The input operator **reads characters from the input stream into a temporary object of type `string`.** It reads until it has read as many characters as the size of the corresponding `bitset`, or it encounters a character other than 1 or 0, or it encounters end-of-file or an input error. **The `bitset` is then initialized from that temporary `string`.**

The output operator prints the bit pattern in a `bitset` object

## 17.3. Regular Expressions

A regular expression is a way of describing a sequence of characters. We’ll focus on how to use the C++ regular-expression library (RE library), which is part of the new library. The RE library, which is defined in the `regex` header, involves several components:

 ![image-20220829163740953](images/image-20220829163740953.png)

## 17.4. Random Numbers

The random-number library, defined in the `random` header, generates random numbers through a set of cooperating classes: **random-number engines** and **random-number distribution** classes.

 ![image-20220830153324232](images/image-20220830153324232.png)

C++ programs should not use the library `rand` function. Instead, they should use the `default_random_engine` along with an appropriate distribution object.

### 17.4.1. Random-Number Engines and Distribution

The random-number engines are function-object classes that define a call operator that takes no arguments and returns a random `unsigned` number.

```c++
default_random_engine e;
for(size_t i = 0; i < 10; ++i)
    cout << e() << " ";
```

Random Number Engine Operations:

 ![image-20220830153749723](images/image-20220830153749723.png)

For most purposes, the output of an engine is not directly usable. The problem is that the numbers usually span a range that differs from the one we need. Correctly transforming the range of a random number is surprisingly hard.

#### Distribution Types and Engines

To get a number in a specified range, we use an object of a distribution type:

```c++
// uniformly distributed from 0 to 9 inclusive
uniform_int_distribution<unsigned> u(0,9);
default_random_engine e; // generates unsigned random integers
for (size_t i = 0; i < 10; ++i)
	// u uses e as a source of numbers
	// each call returns a uniformly distributed value in the specified range
	cout << u(e) << " ";
```

Like the engine types, the distribution types are also function-object classes. The distribution types define a call operator that takes a random-number engine as its argument.

Note that we pass the engine object itself, `u(e)`. Had we written the call as `u(e())`, we would have tried to pass the next value generated by `e` to `u`, which would be a compile-time error.

#### Comparing Random Engines and the `rand` Function

Prior to the new standard, both C and C++ relied on a simple C library function named `rand`. That function produces pseudorandom integers that are uniformly distributed in the range from 0 to a system-dependent maximum value that is at least 32767.

The output of calling a `default_random_engine` object is similar to the output of `rand`. Engines deliver unsigned integers in a system-defined range. The range for `rand` is 0 to `RAND_MAX`.

#### Engines Generate a Sequence of Numbers

Random number generators have one property that often confuses new users: **<font color='red'>Even though the numbers that are generated appear to be random, a given generator returns the same sequence of numbers each time it is run.</font>**

**<font color='red'>As a result, a function with a local random-number generator should make that generator (both the engine and distribution objects) `static`. </font>**Otherwise, the function will generate the identical sequence on each call：

```c++
vector<unsigned> good_randVec(){
    static default_random_engine e;
    static uniform_int_distribution<unsigned> u(0, 9);
    vector<unsigned> ret;
    for(size_t i = 0; i < 100; ++i)
        ret.push_back(u(e));
    return ret;
}
// v1 and v2 have different values
vector<unsigned> v1(good_randVec());
vector<unsigned> v2(good_randVec());
```

Had we written `e` and `u` non`static`, `v1` and `v2` would have the same values.

#### Seeding a Generator

The fact that a generator returns the same sequence of numbers is helpful during debugging. However, once our program is tested, we often want to cause each run of the program to generate different random results. We do so by providing a **<font color='blue'>seed</font>**.

**<font color='red'>A seed is a value that an engine can use to cause it to start generating numbers at a new point in its sequence.</font>**

```c++
default_random_engine e1; 					// uses the default seed
default_random_engine e2(2147483646); 		// use the given seed value
default_random_engine e3; 					// uses the default seed value
e3.seed(32767); 							// call seed to set a new seed value
default_random_engine e4(32767); 			// set the seed value to 32767
```

Here we define four engines. The first two, `e1` and `e2`, have different seeds and should generate different sequences. The second two, `e3` and `e4`, have the same seed value. These two objects will generate the same sequence.

Picking a good seed is surprisingly hard. **<font color='red'>Perhaps the most common approach is to call the system `time` function.</font>** 

This function, defined in the `ctime` header, returns the number of seconds since a given epoch. The `time` function takes a single parameter that is a pointer to a structure into which to write the `time`. If that pointer is `null`, the function just returns the time:

```c++
default_ramdom_engine e(time(0));
```

### 17.4.2. Other Kinds of Distributions

#### Generating Random Real Numbers

Programs often need a source of random floating-point values. In particular, programs frequently need random numbers between zero and one.

The most common, but incorrect, way to obtain a random floating-point from `rand` is to divide the result of r`and()` by `RAND_MAX`. This technique is incorrect because random integers usually have less precision than floating-point numbers, in
which case there are some floating-point values that will never be produced as output.

With the new library facilities, we can easily obtain a floating-point random number. We define an object of type `uniform_real_distribution`:

```c++
default_random_engine e;
uniform_real_distribution<double> u(0, 1);
cout << u(e) << endl;
```

## 17.5. The IO Library Revisited

In this section we’ll look at three of the more specialized features that the IO library supports: format control, unformatted IO, and random access.

### 17.5.1. Formatted Input and Output

In addition to its condition state, each `iostream` object also maintains a **format state** that controls the details of how IO is formatted.

The library defines a set of **<font color='blue'>manipulators 操纵符 </font>** that modify the format state of a stream. A manipulator is a function or object that affects the state of a stream. Like the input and output operators, a manipulator returns the stream object to which it is applied, so we can combine manipulators and data in a single statement.

Our programs have already used one manipulator, `endl`, which we “write” to an output stream as if it were a value. **<font color='red'>But `endl` isn’t an ordinary value; instead, it performs an operation: It writes a newline and flushes the buffer.</font>**

#### Many Manipulators Change the Format State

Most of the manipulators that change the format state provide set/unset pairs; one manipulator sets the format state to a new value and the other unsets it, restoring the normal default formatting.

Manipulators that change the format state of the stream usually leave the format state changed for all subsequent IO. It is usually best to undo whatever state changes are made as soon as those changes are no longer needed.

#### Controlling the Format of Boolean Values

By default, `bool` values print as `1` or `0`. We can override this formatting by applying the `boolalpha` manipulator to the stream:

```c++
cout << boolalpha << true << " " << false << endl;
```

Once we “write” `boolalpha` on `cout`, we’ve changed how `cout` will print `bool` values from this point on. Subsequent operations that print `bool`s will print them as either `true` or `false`.

To undo the format state change to `cout`, we apply `noboolalpha`:

```c++
bool bool_val = get_status();
cout << boolalpha 			// sets the internal state of cout
		<< bool_val
		<< noboolalpha; 	// resets the internal state to default formatting
```

#### Specifying the Base for Integral Values

By default, integral values are written and read in decimal notation. We can change the notational base to octal or hexadecimal or back to decimal by using the manipulators `hex`, `oct`, and `dec`:

```c++
cout << "default: " << 20 << " " << 1024 << endl;
cout << "octal: " << oct << 20 << " " << 1024 << endl;
cout << "hex: " << hex << 20 << " " << 1024 << endl;
cout << "decimal: " << dec << 20 << " " << 1024 << endl;
```

#### Indicating Base on the Output

If we need to print octal or hexadecimal values, it is likely that we should also use the `showbase` manipulator. The `showbase` manipulator causes the output stream to use the same conventions as used for specifying the base of an integral constant:

* A leading 0x indicates hexadecimal.
* A leading 0 indicates octal.
* The absence of either indicates decimal.

```c++
cout << showbase; 											// show the base when printing integral values
cout << "default: " << 20 << " " << 1024 << endl;			// default: 20 1024
cout << "in octal: " << oct << 20 << " " << 1024 << endl;	// in octal: 024 02000
cout << "in hex: " << hex << 20 << " " << 1024 << endl;		// in hex: 0x14 0x400s
cout << noshowbase; 										// reset the state of the stream
```

#### Controlling the Format of Floating-Point Values

We can control three aspects of floating-point output:

* How many digits of precision are printed
* Whether the number is printed in hexadecimal, fixed decimal, or scientific notation
* Whether a decimal point is printed for floating-point values that are whole numbers

1. **Specifying How Much Precision to Print**

   We can change the precision by calling the `precision` member of an IO object or by using the `setprecision` manipulator. 

   * The `precision` member is overloaded. One version takes an `int` value and sets the precision to that new value. It returns the previous precision value. The other version takes no arguments and returns the current precision value. 
   * The `setprecision` manipulator takes an argument, which it uses to set the precision.

   **<font color='red'>By default, precision specifies the total number of digits—both before and after the decimal point.</font>**

   ```c++
   cout << sqrt(2.0) << endl;		// 1.41421
   
   cout.precison(12);
   cout << sqrt(2.0) << endl;		// 1.41421356237
   
   cout << setprecison(3);
   cout << sqrt(2.0) << endl;		// 1.41
   ```

2. **Specifying the Notation of Floating-Point Numbers**

   By default, floating-point values are printed in either fixed decimal or scientific notation depending on the value of the number. The library chooses a format that enhances readability of the number. Very large and very small values are printed using scientific notation. Other values are printed in fixed decimal.

   We can force a stream to use scientific, fixed, or hexadecimal notation by using the appropriate manipulator. The `scientific` manipulator changes the stream to use scientific notation. The `fixed` manipulator changes the stream to use fixed decimal.

   ```c++
   cout << 100 * sqrt(2.0) << endl;				// 141.421
   cout << scientific << 100 * sqrt(2.0) << endl;	// 1.414214e+002
   ```

3. **Printing the Decimal Point**

   By default, when the fractional part of a floating-point value is 0, the decimal point is not displayed. The `showpoint` manipulator forces the decimal point to be printed:

   ```c++
   cout << 10.0 << endl; 		// prints 10
   cout << showpoint << 10.0 	// prints 10.0000
   << noshowpoint << endl; 	// revert to default format for the decimal points
   ```

#### Padding the Output 输出补白

The library provides several manipulators to help us accomplish the control we might need:

* `setw` to specify the minimum space for the next numeric or `string` value.
* `left` to left-justify the output.
* `right` to right-justify the output. Output is right-justified by default.
* `internal` controls placement of the sign on negative values. `internal` left-justifies the sign and right-justifies the value, padding any intervening space with blanks.
* `setfill` lets us specify an alternative character to use to pad the output. By default, the value is a space.

> **<font color='red'>`setw`, like `endl`, does not change the internal state of the output stream. It determines the size of only the next output.</font>**

```c++
int i = -16;
double d = 3.14159;
// pad the first column to use a minimum of 12 positions in the output
cout << "i: " << setw(12) << i << "next col" << '\n'
		<< "d: " << setw(12) << d << "next col" << '\n';
// pad the first column and left-justify all columns
cout << left
		<< "i: " << setw(12) << i << "next col" << '\n'
		<< "d: " << setw(12) << d << "next col" << '\n'
// pad the first column and right-justify all columns
cout << right
		<< "i: " << setw(12) << i << "next col" << '\n'
		<< "d: " << setw(12) << d << "next col" << '\n';
// pad the first column but put the padding internal to the field
cout << internal
		<< "i: " << setw(12) << i << "next col" << '\n'
		<< "d: " << setw(12) << d << "next col" << '\n';
```

When executed, this program generates

 ![image-20220830171542621](images/image-20220830171542621.png)

#### Controlling Input Formatting

By default, the input operators ignore whitespace (blank, tab, newline, formfeed, and carriage return). The `noskipws` manipulator causes the input operator to read, rather than skip, whitespace. To return to the default behavior, we apply the `skipws` manipulator:

```c++
cin >> noskipws; // set cin so that it reads whitespace
while (cin >> ch)
    cout << ch;
cin >> skipws;
```

given the input sequence

```c++
a b  c
d
```

this loop makes seven iterations, reading whitespace as well as the characters in the input. This loop generates:
```c++
a b  c
d
```

### 17.5.2. Unformatted Input/Output Operations

The library also provides a set of low-level operations that support unformatted IO. These operations let us deal with a stream as **a sequence of uninterpreted bytes**.

#### Single-Byte Operations

Several of the unformatted operations deal with a stream **<font color='red'>one byte at a time</font>**. These operations read rather than ignore whitespace.

 ![image-20220830215434648](images/image-20220830215434648.png)

#### Putting Back onto an Input Stream

Sometimes we need to read a character in order to know that we aren’t ready for it. In such cases, we’d like to put the character back onto the stream. The library gives us three ways to do so:

* `peek` returns a copy of the next character on the input stream but does not change the stream. The value returned by `peek` stays on the stream.
* `unget` backs up the input stream so that whatever value was last returned is still on the stream.
* `putback` is a more specialized version of `unget`: It returns the last value read from the stream but takes an argument that must be the same as the one that was last read.

#### `int` Return Values from Input Operations

The `peek` function and the version of `get` that takes no argument return a character from the input stream as an `int`. The reason that these functions return an `int` is to allow them to return an end-of-file marker. These functions convert the character they return to `unsigned char` and then promote that value to `int`. 

**<font color='red'>It is essential that we use an `int` to hold the return from these functions:</font>**

```c++
int ch; // use an int, not a char to hold the return from get()
// loop to read and write all the data in the input
while ((ch = cin.get()) != EOF)
	cout.put(ch);
```

### 17.5.3. Random Access to a Stream

The various stream types generally support random access to the data in their associated stream. We can reposition the stream so that it skips around, reading first the last line, then the first, and so on. 

The library provides a pair of functions to seek to a given location and to tell the current location in the associated stream.

#### Seek and Tell Functions

`istream` and `ostream` types usually do not support random access, the remainder of this section should be considered as applicable to only the `fstream` and `sstream` types.

 ![image-20220830221330032](images/image-20220830221330032.png)

To support random access, the IO types maintain a **marker** that determines where the next read or write will happen. They also provide two functions: One repositions the marker by seeking to a given position; the second tells us the current position of the
marker. 

The library actually defines two pairs of seek and tell functions.. One pair is used by input streams, the other by output streams. The input and output versions are distinguished by a suffix that is either a `g` or a `p`. The `g` versions indicate that we are “getting” (reading) data, and the `p` functions indicate that we are “putting” (writing) data.

#### There Is Only One Marker

The fact that the library distinguishes between the “putting” and “getting” versions of the seek and tell functions can be misleading. Even though the library makes this distinction, it maintains only a single marker in a stream—there is not a distinct read marker and write marker.

**<font color='red'>Because there is only a single marker, we must do a seek to reposition the marker whenever we switch between reading and writing.</font>**

#### Repositioning the Marker

There are two versions of the seek functions: One moves to an “absolute” address within the file; the other moves to a byte offset from a given position:
```c++
// set the marker to a fixed position
seekg(new_position); // set the read marker to the given pos_type location
seekp(new_position); // set the write marker to the given pos_type location
// offset some distance ahead of or behind the given starting point
seekg(offset, from); // set the read marker offset distance from from
seekp(offset, from); // offset has type off_type
```

#### Accessing the Marker

The `tellg` or `tellp` functions return a `pos_type` value denoting the current position of the stream. The tell functions are usually used to remember a location so that we can subsequently seek back to it:

```c++
ostringstream writeStr;
auto mark = writeStr.tellp();
// ...
if(cancelEntry)
    // return to the remembered position
    writeStr.seekp(mark);
```

#### Reading and Writing to the Same File

Let’s look at a programming example. Assume we are given a file to read. We are to write a newline at the end of the file that contains the relative position at which each line begins. For example, given the following file,

```tex
abcd
efg
hi
j
```

the program should produce the following modified file:

```tex
abcd
efg
hi
j
5 9 12 14
```

1. Note that our program need not write the offset for the first line—it always occurs at position 0. 

2. Also note that the offset counts must include the invisible newline character that ends each line. 

3. Finally, note that the last number in the output is the offset for the line on which our output begins. 

```c++
int main()
{
	// open for input and output and preposition file pointers to end-of-file
	fstream inOut("copyOut", fstream::ate | fstream::in | fstream::out);
	if (!inOut) {
		cerr << "Unable to open file!" << endl;
		return EXIT_FAILURE;
	}
	// inOut is opened in ate mode, so it starts out positioned at the end
	auto end_mark = inOut.tellg();				// remember original end-of-file position
	inOut.seekg(0, fstream::beg); 				// reposition to the start of the file
	size_t cnt = 0; 							// accumulator for the byte count
	string line; 								// hold each line of input
	while (inOut && inOut.tellg() != end_mark && getline(inOut, line)) {
		cnt += line.size() + 1; 				// add 1 to account for the newline
		auto mark = inOut.tellg(); 				// remember the read position
		inOut.seekp(0, fstream::end); 			// set the write marker to the end
		inOut << cnt;	 						// write the accumulated length
		if (mark != end_mark)					// print a separator if this is not the last line
            inOut << " ";
		inOut.seekg(mark); 						// restore the read position
	}
	inOut.seekp(0, fstream::end); 				// seek to the end
	inOut << "\n"; 								// write a newline at end-offile
	return 0;
}
```

# Chapter 18. Tools for Large Programs

## 18.1. Exception Handling

Exceptions let us separate problem detection from problem resolution. One part of the program can detect a problem and can pass the job of resolving that problem to another part of the program. The detecting part need not know anything about the handling part, and vice versa.

### 18.1.1. Throwing an Exception

In C++, an exception is raised by **<font color='blue'>throwing an expression</font>**. The type and contents of that object allow the throwing part of the program to inform the handling part about what went wrong.

When a `throw` is executed, the statement(s) following the `throw` are not executed. Instead, control is transferred from the `throw` to the matching `catch`.

Because the statements following a `throw` are not executed, a `throw` is like a `return`: It is usually part of a conditional statement or is the last (or only) statement in a function.

#### Stack Unwinding 栈展开

When an exception is thrown, execution of the current function is suspended and the search for a matching `catch` clause begins. If the `throw` appears inside a `try` block, the catch clauses associated with that `try` are examined:

* If a matching `catch` is found, the exception is handled by that `catch`. 
* Otherwise, if the `try` was itself nested inside another `try`, the search continues through the `catch` clauses of the enclosing `try`s. If no matching `catch` is found, the current function is exited, and the search continues in the calling function.

If the call to the function that threw is in a `try` block, then the `catch` clauses associated with that `try` are examined:

* If a matching `catch` is found, the exception is handled. 
* Otherwise, if that `try` was nested, the `catch` clauses of the enclosing `try`s are searched. If no `catch` is found, the calling function is also exited. 

The search continues in the function that called the just exited one, and so on.

This process, known as **<font color='blue'>stack unwinding</font>**, continues up the chain of nested function calls until a catch clause for the exception is found, or the main function itself is exited without having found a matching `catch`.

Assuming a matching `catch` is found, that `catch` is entered. **<font color='red'>When the `catch` completes, execution continues at the point immediately after the last `catch` clause associated with that `try` block.</font>**

If no matching `catch` is found, the program calls the library `terminate` function. As its name implies, `terminate` stops execution of the program.

#### Objects Are Automatically Destroyed during Stack Unwinding

When a block is exited during stack unwinding, the compiler guarantees that objects created in that block are properly destroyed. If a local object is of class type, the destructor for that object is called automatically. As usual, the compiler does no work to destroy objects of built-in type.

#### Destructors and Exceptions

The fact that destructors are run during stack unwinding affects how we write destructors:

 During stack unwinding, an exception has been raised but is not yet handled.**<font color='red'> If a new exception is thrown during stack unwinding and not caught in the function that threw it, `terminate` is called.</font>** 

Because destructors may be invoked during stack unwinding, they should never throw exceptions that the destructor itself does not handle. That is,**<font color='red'> if a destructor does an operation that might throw, it should wrap that operation in a `try` block and handle it locally to the destructor.</font>**

#### The Exception Object

The compiler uses the thrown expression to copy initialize a special object known as the exception object. As a result, the expression in a `throw` must have a complete type.

The exception object resides in space, managed by the compiler, that is guaranteed to be accessible to whatever `catch` is invoked. The exception object is destroyed after the exception is completely handled.

When we throw an expression, the static, compile-time type of that expression determines the type of the exception object. If a `throw` expression dereferences a pointer to a base-class type, and that pointer points to a derived-type object, then the thrown object is sliced down only the base-class part is thrown.

### 18.1.2. Catching an Exception

The **<font color='blue'>exception declaration</font>** in a `catch` clause looks like a function parameter list with exactly one parameter. As in a parameter list, we can omit the name of the `catch` parameter if the `catch` has no need to access the thrown expression.

When a `catch` is entered, the parameter in its exception declaration is initialized by the exception object. Also like a function parameter, a `catch` parameter that has a base-class type can be initialized by an exception object that has a type derived from the parameter type：

* If the `catch` parameter has a nonreference type, then the exception object will be sliced down.
* On the other hand, if the parameter is a reference to a base-class type, then the parameter is bound to the exception object in the usual way

**<font color='red'>Ordinarily, a `catch` that takes an exception of a type related by inheritance ought to define its parameter as a reference.</font>**

#### Finding a Matching Handler

During the search for a matching `catch`, the `catch` that is found is not necessarily the one that matches the exception best. Instead, the selected `catch` is **the first one** that matches the exception at all. Thus programs that use exceptions from an inheritance hierarchy must order their `catch` clauses so that **<font color='red'>handlers for a derived type occur before a `catch` for its base type.</font>**

#### Rethrow

A `catch` passes its exception out to another `catch` by **<font color='blue'>rethrowing</font>** the exception. A rethrow is a `throw` that is not followed by an
expression:

```c++
throw;
```

A rethrow does not specify an expression; the (current) exception object is passed up the chain.

**<font color='red'>In general, a `catch` might change the contents of its parameter. If, after changing its parameter, the `catch` rethrows the exception, then those changes will be propagated only if the `catch`’s exception declaration is a reference</font>**：

```c++
catch (my_error &eObj) { 				// specifier is a reference type
	eObj.status = errCodes::severeErr; 	// modifies the exception object
	throw; 								// the status member of the exception object is severeErr
} catch (other_error eObj) { 			// specifier is a nonreference type
	eObj.status = errCodes::badErr; 	// modifies the local copy only
	throw; 								// the status member of the exception object is unchanged
}
```

#### The Catch-All Handler

To catch all exceptions, we use an ellipsis for the exception declaration. Such handlers, sometimes known as catch-all handlers, have the form `catch(...)`

A `catch(...)` is often used in combination with a `rethrow` expression. The `catch` does whatever local work can be done and then rethrows the exception:

```c++
void manip() {
	try {
		// actions that cause an exception to be thrown
	}
	catch (...) {
		// work to partially handle the exception
		throw;
	}
}
```

### 18.1.3. Function `try` Blocks and Constructors

An exception might occur while processing a **constructor initializer**. Constructor initializers execute before the constructor body is entered. A `catch` inside the constructor body can’t handle an exception thrown by a constructor initializer. To handle an exception from a constructor initializer, we must write the constructor as a **<font color='blue'>function `try` block</font>**.

As an example, we might wrap the `Blob` constructors in a function `try` block:

```c++
template <typename T>
Blob<T>::Blob(initializer_list<T> il) try :
				data(make_shared<vector<T>>(il)){
	/* empty function body */                    
}
catch(const bad_alloc &e){
    handle_out_of_memory(e);
}
```

Notice that the keyword `try` appears before the colon that begins the constructor initializer list and before the curly brace that forms the (in this case empty) constructor function body.

The `catch` associated with this `try` can be used to handle exceptions thrown either from within the member initialization list or from within the constructor body.

### 18.1.4. The `noexcept` Exception Specification

Under the new standard, a function can specify that it does not throw exceptions by providing a `noexcept` specification:

```c++
void recoup(int) noexcept; // won't throw
void alloc(int); // might throw
```

The `noexcept` specifier must appear on all of the declarations and the corresponding definition of a function or on none of them.

In a member function the `noexcept` specifier follows any `const` or `reference` qualifiers, and it precedes `final`, `override`, or   `= 0` on a virtual function.

#### Violating the Exception Specification

It is important to understand that the compiler does not check the `noexcept` specification at compile time. As a result, it is possible that a function that claims it will not throw will in fact throw:

```c++
// this function will compile, even though it clearly violates its exception specification
void f() noexcept 		// promises not to throw any exception
{
	throw exception(); 	// violates the exception specification
}
```

If a `noexcept` function does throw, `terminate` is called.

#### Arguments to the `noexcept` Specification

The `noexcept` specifier takes an optional argument that must be convertible to `bool`: 

* If the argument is `true`, then the function won’t throw
* If the argument is `false`, then the function might throw

```c++
void recoup(int) noexcept(true); // recoup won't throw
void alloc(int) noexcept(false); // alloc can throw
```

#### The `noexcept` Operator

The `noexcept` operator is a unary operator that returns a `bool` rvalue constant expression that indicates whether a given expression might throw:

```c++
noexcept(recoup(i)) // true if calling recoup can't throw, false otherwise
```

We can use the `noexcept` operator to form an exception specifier as follows:

```c++
void f() noexcept(noexcept(g()));	// f has same exception specifier as g
```

#### Exception Specifications and Pointers, Virtuals, and Copy Control

1. If we declare a function pointer that has a nonthrowing exception specification, we can use that pointer only to point to nonthrowing functions. A pointer that specifies (explicitly or implicitly) that it might throw can point to any function.

   ```c++
   // both recoup and pf1 promise not to throw
   void (*pf1)(int) noexcept = recoup;
   // ok: recoup won't throw; it doesn't matter that pf2 might
   void (*pf2)(int) = recoup;
   
   pf1 = alloc; // error: alloc might throw but pf1 said it wouldn't
   pf2 = alloc; // ok: both pf2 and alloc might throw
   ```

2. If a virtual function includes a promise not to throw, the inherited virtuals must also promise not to throw. On the other hand, if the base allows exceptions, it is okay for the derived functions to be more restrictive and promise not to throw

3. When the compiler synthesizes the copy-control members, it generates an exception specification for the synthesized member. If all the corresponding operation for all the members and base classes promise not to throw, then the synthesized member is `noexcept`.

### ⭐18.1.5. Exception Class Hierarchies

The standard-library exception classes form the inheritance hierarchy：

 ![image-20220831161414625](images/image-20220831161414625.png)

The only operations that the `exception` types define are the copy constructor, copy-assignment operator, a virtual destructor, and a virtual member named `what`.

The `exception`, `bad_cast`, and `bad_alloc` classes also define a default constructor. The `runtime_error` and `logic_error` classes do not have a default constructor but do have constructors that take a C-style character string or a library `string` argument.

#### Exception Classes for a Bookstore Application

Applications often extend the `exception` hierarchy by defining classes derived from `exception` (or from one of the library classes derived from `exception`).

If we were building a real bookstore application, we probably would have defined our own hierarchy of exceptions to represent application-specific problems：

```c++
class out_of_stock : public runtime_error{
public:
    explicit out_of_stock(const string &s): runtime_error(s) { }
};
class isbn_mismatch: public logic_error {
public:
	explicit isbn_mismatch(const string &s): logic_error(s) { }
	isbn_mismatch(const string &s, const string &lhs, const string &rhs):
						logic_error(s), left(lhs), right(rhs) { }
	const string left, right;
};
```

Our application-specific exception types inherit them from the standard `exception` classes. As with any hierarchy, we can think of the `exception` classes as being organized into layers. As the hierarchy becomes deeper, each layer becomes a more specific exception.

#### Using Our Own Exception Types

We use our own exception classes in the same way that we use one of the standard library classes.

```c++
// throws an exception if both objects do not refer to the same book
Sales_data& Sales_data::operator+=(const Sales_data &rhs){
    if(isbn() != rhs.isbn())
        throw isbn_mismatch("wrong isbns", isbn(), rhs.isbn());
    units_sold += rhs.units_sold;
	revenue += rhs.revenue;
	return *this;
}
```

Code that uses the compound addition operator can detect this error, write an appropriate error message, and continue:

```c++
Sales_data item1, item2, sum;
while (cin >> item1 >> item2) { 	// read two transactions
	try {
		sum = item1 + item2; // calculate their sum
		// use sum
	} catch (const isbn_mismatch &e) {
		cerr << e.what() << ": left isbn(" << e.left
				<< ") right isbn(" << e.right << ")" << endl;
	}
}
```

## 18.2. Namespaces

### 18.2.1. Namespace Definitions

A namespace definition begins with the keyword `namespace` followed by the namespace name.

Namespaces may be defined at global scope or inside another namespace. They may not be defined inside a function or a class.

A namespace scope does not end with a semicolon.

```c++
namespace cplusplus_primer {
	class Sales_data { / * ... * /};
	Sales_data operator+(const Sales_data&, const Sales_data&);
	class Query { /* ... */ };
	class Query_base { /* ... */};
} // like blocks, namespaces do not end with a semicolon
```

#### Each Namespace Is a Scope

Names defined in a namespace may be accessed directly by other members of the namespace, including scopes nested within those members. Code outside the namespace must indicate the namespace in which the name is defined:

```c++
cplusplus_primer::Query q = cplusplus_primer::Query("hello");
```

#### ⭐Namespaces Can Be Discontiguous

Unlike other scopes, a namespace can be defined in several parts. Writing a namespace definition:

```c++
namespace nsp {
	// declarations
}
```

either defines a new namespace named `nsp` or adds to an existing one.

The fact that namespace definitions can be discontiguous lets us compose a namespace from separate interface and implementation files：

* Namespace members that define classes, and declarations for the functions and objects that are part of the class interface, can be put into header files. 
* The definitions of namespace members can be put in separate source files.

#### Defining the Primer Namespace

Using this strategy for separating interface and implementation, we might define the `cplusplus_primer` library in several separate files：

```c++
// ---- Sales_data.h----
// #includes should appear before opening the namespace
#include <string>
namespace cplusplus_primer {
	class Sales_data { /* ... */};
	Sales_data operator+(const Sales_data&, const Sales_data&);
	// declarations for the remaining functions in the Sales_data interface
}

// ---- Sales_data.cc----
// be sure any #includes appear before opening the namespace
#include "Sales_data.h"
namespace cplusplus_primer {
	// definitions for Sales_data members and overloaded operators
}
```

A program using our library would include whichever headers it needed. The names in those headers are defined inside the `cplusplus_primer` namespace:

```c++
// ---- user.cc----
// names in the Sales_data.h header are in the cplusplus_primer namespace
#include "Sales_data.h"
int main()
{
	using cplusplus_primer::Sales_data;
	Sales_data trans1, trans2;
	// ...
	return 0;
}
```

It is worth noting that ordinarily, we do not put a `#include` inside the namespace. If we did, we would be attempting to define all the names in that header as members of the enclosing namespace.

#### Template Specializations

Template specializations must be defined in the same namespace that contains the original template. As with any other namespace name, so long as we have declared the specialization inside the namespace, we can define it outside the
namespace:

```c++
// we must declare the specialization as a member of std
namespace std{
    template <> struct hash<Sales_data>;
}
// having added the declaration for the specialization to std
// we can define the specialization outside the std namespace
template <> struct std::hashs<Sales_data>{
	size_t operator()(const Sales_data& s) const{
        return hash<string>()(s.bookNo) ^
				hash<unsigned>()(s.units_sold) ^
				hash<double>()(s.revenue);
    }
	// other members as before  
};
```

#### The Global Namespace

Names defined at global scope (i.e., names declared outside any class, function, or namespace) are defined inside the global namespace. The global namespace is implicitly declared and exists in every program. Each file that defines entities at global
scope (implicitly) adds those names to the global namespace.

The scope operator can be used to refer to members of the global namespace. Because the global namespace is implicit, it does not have a name; the notation

```c++
::member_name
```

refers to a member of the global namespace.

#### Nested Namespaces

A nested namespace is a namespace defined inside another namespace. 

Nested namespace names follow the normal rules: Names declared in an inner namespace hide declarations of the same name in an outer namespace:

```c++
namespace cplusplus_primer {
	// first nested namespace: defines the Query portion of the library
	namespace QueryLib {
		class Query { /* ... */ };
		Query operator&(const Query&, const Query&);
		// ...
	}
	// second nested namespace: defines the Sales_data portion of the library
	namespace Bookstore {
		class Quote { /* ... */ };
		class Disc_quote : public Quote { /* ... */ };
		// ...
	}
}
```

**<font color='red'>Code in the outer parts of the enclosing namespace may refer to a name in a nested namespace only through its qualified name:</font>**

```c++
cplusplus_primer::QueryLib::Query
```

#### Inline Namespaces

The new standard introduced a new kind of nested namespace, an inline namespace. Unlike ordinary nested namespaces, names in an inline namespace can be used as if they were direct members of the enclosing namespace.

We need not qualify names from an inline namespace by their namespace name. We can access them using only the name of the enclosing namespace:

```c++
namespace cplusplus_primer {
	inline namespace FifthEd {
		class Query_base { /* ... */ };
		// other Query-related declarations
	}
	namespace FourthEd {
		class Item_base { /* ... */};
		class Query_base { /* ... */};
		// other code from the Fourth Edition
	}
}
```

Because `FifthEd` is inline, code that refers to `cplusplus_primer::` will get the version from that namespace. If we want the earlier edition code, we can access it as we would any other nested namespace, by using the names of all the enclosing
namespaces: for example, `cplusplus_primer::FourthEd::Query_base`.

#### Unnamed Namespaces

An unnamed namespace is the keyword `namespace` followed immediately by a block of declarations delimited by curly braces. Variables defined in an unnamed namespace have static lifetime: They are created before their first use and destroyed
when the program ends.

An unnamed namespace may be discontiguous within a given file but does not span files. Each file has its own unnamed namespace. If two files contain unnamed namespaces, those namespaces are unrelated.

Names defined in an unnamed namespace are used directly; after all, there is no namespace name with which to qualify them.

Names defined in an unnamed namespace are in the same scope as the scope at which the namespace is defined. If an unnamed namespace is defined at the outermost scope in the file, then names in the unnamed namespace must differ from names defined at global scope:

```c++
int i; // global declaration for i
namespace {
	int i;
}
// ambiguous: defined globally and in an unnested, unnamed namespace
i = 10;
```

### 18.2.2. Using Namespace Members
