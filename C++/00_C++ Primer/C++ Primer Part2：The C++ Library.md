# Chapter 8. The IO Library

## 8.1. The IO Classes

The library defines a collection of IO types in addition to the `istream` and `ostream` types that we have already used.

 ![image-20220718202424243](images/image-20220718202424243.png)

To support languages that use wide characters, the library defines a set of types and objects that manipulate `wchar_t` data. The names of the widecharacter versions begin with a `w`.

#### Relationships among the IO Types

The types `ifstream` and `istringstream` inherit from `istream`. Thus, we can use objects of type `ifstream` or `istringstream` as if they were `istream` objects. We can use objects of these types in the same ways as we have used `cin`. For example, we can call `getline` on an `ifstream` or `istringstream` object, and we can use the `>>` to read data from an `ifstream` or `istringstream`. 

Similarly, the types `ofstream` and `ostringstream` inherit from `ostream`. Therefore, we can use objects of these types in the same ways that we have used `cout`.

### 8.1.1. No Copy or Assign for IO Objects

Because we can’t copy the IO types, we cannot have a parameter or return type that is one of the stream types. Functions that do IO typically pass and
return the stream through references. Reading or writing an IO object changes its state, so the reference must not be `const`.

### 8.1.2. Condition States

The IO classes define functions and flags, listed in the table below, that let us access and manipulate the condition state of a stream.

 ![image-20220718203423011](images/image-20220718203423011.png)

Once an error has occurred, subsequent IO operations on that stream will fail. We can read from or write to a stream only when it is in a non-error state.

#### Interrogating the State of a Stream

The IO library defines a machine-dependent integral type named `iostate` that it uses to convey information about the state of a stream. This type is used as a collection of bits. The IO classes define four `constexpr` values of type `iostate` that represent particular bit patterns：

* The `badbit` indicates a system-level failure, such as an unrecoverable read or write error. It is usually not possible to use a stream once `badbit` has been set.
* The `failbit` is set after a recoverable error, such as reading a character when numeric data was expected. It is often possible to correct such problems and continue using the stream.
* Reaching end-of-file sets both `eofbit` and `failbit`.
* The `goodbit`, which is guaranteed to have the value `0`, indicates no failures on the stream.

The library also defines a set of functions to interrogate the state of these flags：

* The `good` operation returns `true` if none of the error bits is set.
* The `bad`, `fail`, and `eof` operations return `true` when the corresponding bit is on.
* In addition, `fail` returns `true` if `bad` is set.

**<font color='red'>By implication, the right way to determine the overall state of a stream is to use either `good` or `fail`.</font>**

#### Managing the Condition State

The `rdstate` member returns an `iostate` value that corresponds to the current state of the stream. 

The `setstate` operation turns on the given condition bit(s) to indicate that a problem occurred.

The version of `clear` that takes no arguments turns off all the failure bits. After `clear()`, a call to `good` returns `true`. 

The version of `clear` that takes an argument expects an `iostate` value that represents the new state of the stream.

```c++
auto old_state = cin.rdstate();		// remember the current state of cin
cin.clear();						// make cin valid
process_input(cin);					// use cin
cin.setstate(old_state);			// now reset cin to its old state
```

To turn off a single condition, we use the `rdstate` member and the bitwise operators to produce the desired new state.

```c++
// turns off failbit and badbit but all other bits unchanged
cin.clear(cin.rdstate() & ~cin.failbit & )
```

###  ⭐8.1.3. Managing the Output Buffer

**<font color='red'>Each output stream manages a buffer,</font>** which it uses to hold the data that the program reads and writes.

There are several conditions that cause the buffer to be flushed—that is, to be written—to the actual output device or file:

* The program completes normally. All output buffers are flushed as part of the `return` from `main`.
* We can flush the buffer explicitly using a manipulator such as `endl`
* We can use the `unitbuf` manipulator to set the stream’s internal state to empty the buffer after each output operation. By default, `unitbuf` is set for `cerr`, so that writes to `cerr` are flushed immediately.
* An output stream might be tied to another stream. In this case, the buffer of the tied stream is flushed whenever the tied stream is read or written. By default, `cin` and `cerr` are both tied to `cout`. Hence, reading `cin` or writing to `cerr` flushes the buffer in `cout`.

#### Flushing the Output Buffer

In addition to `endl` manipulator, There are two other similar manipulators: `flush` and `ends`. `flush` flushes the stream but adds no characters to the output; `ends` inserts a null character into the buffer and then flushes it:

```c++
cout << "hi!" << endl; 		// writes hi and a newline, then flushes the buffer
cout << "hi!" << flush; 	// writes hi, then flushes the buffer; adds no data
cout << "hi!" << ends; 		// writes hi and a null, then flushes the buffer
```

#### The `unitbuf` Manipulator

If we want to flush after every output, we can use the `unitbuf` manipulator. This manipulator tells the stream to do a flush after every subsequent write. The `nounitbuf` manipulator restores the stream to use normal, system-managed buffer flushing:

```c++
cout << unitbuf; 	// all writes will be flushed immediately
// any output is flushed immediately, no buffering
cout << nounitbuf; 	// returns to normal buffering
```

#### Tying Input and Output Streams Together

When an input stream is tied to an output stream, any attempt to read the input stream will first flush the buffer associated with the output stream. 

**<font color='red'>The library ties `cin` to `cout`</font>**, so the statement `cin >> ival;` causes the buffer associated with `cout` to be flushed.

The function `tie` takes a pointer to an `ostream` and ties itself to that `ostream`. That is, `x.tie(&o)` ties the stream `x` to the output stream `o`.

We can tie either an `istream` or an `ostream` object to another `ostream`:

```c++
cin.tie(&cout);							// illustration only: the library ties cin and cout for us
// old_tie points to the stream (if any) currently tied to cin
ostream *old_tie = cin.tie(nullptr);	// cin is no longer tied
// ties cin and cerr; not a good idea because cin should be tied to cout
cin.tie(&cerr);							// reading cin flushes cerr, not cout
cin.tie(old_tie);						// reestablish normal tie between cin and cout
```

To tie a given stream to a new output stream, we pass `tie` a pointer to the new stream. To untie the stream completely, we pass a null pointer. Each stream can be tied to at most one stream at a time. However, multiple streams can tie themselves to the same `ostream`.

## 8.2. File Input and Output

The `fstream` header defines three types to support file IO. In addition to the behavior that they inherit from the `iostream` types, they add members to manage the file associated with the stream. These operations, listed in the table bellow, can be called on objects of `fstream`, `ifstream`, or `ofstream` but not on the other IO types.

 ![image-20220718211812466](images/image-20220718211812466.png)

#### 8.2.1. Using File Stream Objects

When we want to read or write a file, we define a file stream object and associate that object with the file. When we create a file stream, we can (optionally) provide a file name. When we supply a file name, `open` is called automatically:

```c++
ifstream in(ifile); 	// construct an ifstream and open the given file
ofstream out; 			// output file stream that is not associated with any file
```

#### Using an `fstream` in Place of an `iostream&`

Functions that are written to take a reference (or pointer) to one of the `iostream` types can be called on behalf of the corresponding `fstream` (or `sstream`) type.

For example, we can use the `read` and `print` functions from § 7.1.3 to read from and write to named files.

#### The `open` and `close` Members

When we define an empty file stream object, we can subsequently associate that object with a file by calling `open`. If a call to `open` fails, `failbit` is set. Because a call to `open` might fail, it is usually a good idea to verify that the `open` succeeded

```c++
ifstream in(ifile);
ofstream out();
out.open(ifile + ".copy");
if(out){
    // ...
}
```

Once a file stream has been opened, it remains associated with the specified file. To associate a file stream with a different file, we must first close the existing file. Once the file is closed, we can open a new one:

```c++
out.close();
out.open(ifile + "2");
```

**<font color='red'>When an `fstream` object is destroyed, `close` is called automatically.</font>**

### 8.2.2. File Modes

Each stream has an associated file mode that represents how the file may be used.

 ![image-20220718213505682](images/image-20220718213505682.png)

#### Opening a File in out Mode Discards Existing Data

By default, when we open an `ofstream`, the contents of the file are discarded. The only way to prevent an `ostream` from emptying the given file is to specify `app`:

```c++
// file1 is truncated in each of these cases
ofstream out("file1"); 						// out and trunc are implicit
ofstream out2("file1", ofstream::out); 		// trunc is implicit
ofstream out3("file1", ofstream::out | ofstream::trunc);

// to preserve the file's contents, we must explicitly specify app mode
ofstream app("file2", ofstream::app); 		// out is implicit
ofstream app2("file2", ofstream::out | ofstream::app);
```

## 8.3. string Streams

The `sstream` header defines three types to support in-memory IO; these types read from or write to a `string` as if the `string` were an IO stream.

In addition to the operations they inherit, the types defined in `sstream` add members to manage the `string` associated with the stream.:

 ![image-20220718214226838](images/image-20220718214226838.png)

### ⭐8.3.1. Using an `istringstream`

An `istringstream` is often used when we have some work to do on an entire line, and other work to do with individual words within a line.

As one example, assume we have a file that lists people and their associated phone numbers. Some people have only one number, but others have several—a home phone, work phone, cell number, and so on. Our input file might look like the following:

```te
morgan 2015552368 8625550123
drew 9735550130
lee 6095550132 2015550175 8005550000
```

We’ll start by defining a simple class to represent our input data:

```c++
struct PersonInfo {
	string name;
	vector<string> phones;
};
```

Our program will read the data file and build up a `vector` of `PersonInfo`.

```c++
string line, word; 				// will hold a line and word from input, respectively
vector<PersonInfo> people; 		// will hold all the records from the input
// read the input a line at a time until cin hits end-of-file (or another error)
while (getline(cin, line)) {
	PersonInfo info; 			// create an object to hold this record's data
	istringstream record(line); // bind record to the line we just read
	record >> info.name;	 	// read the name
	while (record >> word) 		// read the phone numbers
		info.phones.push_back(word); // and store them
	people.push_back(info); 	// append this record to people
}
```

We use `getline` to read an entire record from the standard input. If the call to `getline` succeeds, then `line` holds a record from the input file.

We bind an `istringstream` to the `line` that we just read. We can now use the `input` operator on that `istringstream` to read each element in the current record.

### 8.3.2. Using `ostringstream`s

An `ostringstream` is useful when we need to build up our output a little at a time but do not want to print the output until later.

For example, we might want to validate and reformat the phone numbers we read in the previous example. If all the numbers are valid, we want to print a new file containing the reformatted numbers. If a person has any invalid numbers, we won’t put them in the new file. Instead, we’ll write an error message containing the person’s name and a list of their invalid numbers.

```c++
for(const auto &entry : people){
    ostringstream formatted, badNums;
    for(const auto &nums : entry.phones){
        if(!valid(nums))
            badNums << " " << nums;
        else
            formatted << " " << format(nums);
    }
    if(badNums.str().empty())
        os << entry.name << " "
        	<< formatted.str() << endl;
    else
        cerr << "input error: " << entry.name
        		<< " invalid number(s) " << badNums.str() << endl;
}
```

The interesting part of the program is the use of the string streams `formatted` and `badNums`. We use the normal output operator (`<<`) to write to these objects. But, these “writes” are really `string` manipulations. They add characters to the `string`s inside `formatted` and `badNums`, respectively.

# Chapter 9. Sequential Containers

## 9.1. Overview of the Sequential Containers

The sequential containers, which are listed in the table, all provide fast sequential access to their elements.

 ![image-20220720095204083](images/image-20220720095204083.png)

A `deque` is a more complicated data structure. Like `string` and `vector`, **<font color='red'>`deque` supports fast random access.</font>** As with `string` and `vector`, adding or removing elements in the middle of a `deque` is a (potentially) expensive operation. However, adding or removing elements at either end of the `deque` is a fast operation, comparable to adding an element to a `list` or `forward_list`.

The `forward_list` and `array` types were added by the new standard. 

* An `array` is a safer, easier-to-use alternative to built-in arrays. Like built-in arrays, library `array`s have fixed size. As a result, `array` does not support operations to add and remove elements or to resize the container. 
* A `forward_list` is intended to be comparable to the best handwritten, singly linked list. Consequently, `forward_list` does not have the `size` operation because storing or computing its `size` would entail overhead compared to a handwritten list. 

#### Deciding Which Sequential Container to Use

There are a few rules of thumb that apply to selecting which container to use:

* **<font color='red'>Unless you have a reason to use another container, use a `vector`.</font>**
* If your program has lots of small elements and space overhead matters, don’t use `list` or `forward_list`.
* If the program requires random access to elements, use a vector or a `deque`.
* If the program needs to insert or delete elements in the middle of the container,
  use a list or forward_list.
* If the program needs to insert or delete elements at the front and the back, but
  not in the middle, use a deque.
* **<font color='red'>If the program needs to insert elements in the middle of the container only while reading input, and subsequently needs random access to the elements:</font>**
  * First, decide whether you actually need to add elements in the middle of a container. It is often easier to append to a `vector` and then call the library `sort` function to reorder the container when you’re done with input.
  * If you must insert into the middle, consider using a `list` for the input phase. Once the input is complete, copy the `list` into a `vector`.

## 9.2. Container Library Overview

Firstly, we will cover the operations provided by all container types:

 ![image-20220720100707532](images/image-20220720100707532.png)

 ![image-20220720100815006](images/image-20220720100815006.png)

#### Constraints on Types That a Container Can Hold

Almost any type can be used as the element type of a sequential container. But some container operations impose requirements of their own on the element type. We can define a container for a type that does not support an operation-specific requirement, but we can use an operation only if the element type meets that operation’s requirements.

As an example, the sequential container constructor that takes a size argument uses the element type’s default constructor. Some classes do not have a
default constructor. We can define a container that holds objects of such types, but we cannot construct such containers using only an element count:

```c++
// assume noDefault is a type without a default constructor
vector<noDefault> v1(10, init); 	// ok: element initializer supplied
vector<noDefault> v2(10); 			// error: must supply an element initializer
```

### 9.2.1. Iterators

With one exception, the container iterators support all the operations listed in 3.4.1. The exception is that the `forward_list` iterators do not support the
decrement (`--`) operator. The iterator arithmetic operations listed in Table 3.4.2 apply only to iterators for `string`, `vector`, `deque`, and `array`. We cannot use these operations on iterators for any of the other container types.

#### Iterator Ranges

An **<font color='blue'>iterator range</font>** is denoted by a pair of iterators each of which refers to an element, or to one past the last element, in the same container. These two iterators, often referred to as `begin` and `end`.

This element range is called a **<font color='blue'>left-inclusive interval</font>**. The standard mathematical notation for such a range is
$$
[ begin, end)
$$
The iterators `begin` and `end` must refer to the same container. The iterator `end` may be equal to `begin` but must not refer to an element before the one denoted by `begin`.

#### Programming Implications of Using Left-Inclusive Ranges 使用左闭合范围蕴含的编程假设

The library uses left-inclusive ranges because such ranges have three convenient properties:

* If `begin` equals `end`, the range is empty.

* If `begin` is not equal to `end`, there is at least one element in the range, and `begin` refers to the first element in that range.

* We can increment `begin` some number of times until `begin == end`

  ```c++
  while(begin != end){
      *begin = val;
      ++begin;
  }
  ```

### 9.2.2. Container Type Members

Each container defines several types:

 ![image-20220720103635083](images/image-20220720103635083.png)

If we need the element type, we refer to the container’s `value_type`. If we need a reference to that type, we use `reference` or `const_reference`.

To use one of these types, we must name the class of which they are a member:

```c++
list<string>::iterator iter;
vector<int>::difference_type count;
```

### 9.2.3. `begin` and `end` Members

 ![image-20220720103710511](images/image-20220720103710511.png)

The `begin` and `end` operations yield iterators that refer to the first and one past the last element in the container.

There are several versions of `begin` and `end`: The versions with an `r` return reverse iterators. Those that start with a `c` return the const version of the related iterator.

**<font color='red'>The functions that do not begin with a `c` are overloaded. That is, there are actually two members named `begin`. </font>**

* One is a `const` member that returns the container’s `const_iterator` type. 

* The other is non`const` and returns the container’s `iterator` type. 

Similarly for `rbegin`, `end`, and `rend`. When we call one of these members on a non`const` object, we get the version that returns `iterator`. We get a `const` version of the iterators only when we call these functions on a `const` object. 

As with pointers and references to `const`, we can convert a plain `iterator` to the corresponding `const_iterator`, but not vice versa.

```c++
// type is explicitly specified
list<string>::iterator it5 = a.begin();
list<string>::const_iterator it6 = a.begin();
// iterator or const_iterator depending on a's type of a
auto it7 = a.begin(); 		// const_iterator only if a is const
auto it8 = a.cbegin(); 		// it8 is const_iterator
```

### 9.2.4. Defining and Initializing a Container

#### ⭐Initializing a Container as a Copy of Another Container

There are two ways to create a new container as a copy of another one:

* We can directly copy the container (excepting array)

  The container and element types must match.

* We can copy a range of elements denoted by a pair of iterators

  There is no requirement that the container types be identical. Moreover, the element types in the new and original containers can differ as long as it is possible to convert the elements we’re copying to the element type of the container we are initializing

```c++
list<string> authors = {"Milton", "Shakespeare", "Austen"};
vector<const char*> articles = {"a", "an", "the"};
list<string> list2(authors); 					// ok: types match
deque<string> authList(authors);				// error: container types don't match
vector<string> words(articles);					// error: element types must match
forward_list<string> words(articles.begin(), articles.end());	// ok: converts const char* elements to string
```

#### List Initialization

Under the new standard, we can list initialize a container:

```c++
list<string> authors = {"Milton", "Shakespeare", "Austen"};
vector<const char*> articles = {"a", "an", "the"};
```

When we do so, we explicitly specify values for each element in the container. For types other than `array`, the initializer list also implicitly specifies the size of the container: The container will have as many elements as there are initializers.

#### Sequential Container Size-Related Constructors

In addition to the constructors that sequential containers have in common with associative containers, we can also initialize the sequential containers (other than `array`) from a size and an (optional) element initializer. If we do not supply an element initializer, the library creates a **<font color='red'>value-initialized</font>** one for us:

```c++
vector<int> ivec(10, -1); 		// ten int elements, each initialized to -1
list<string> svec(10, "hi!"); 	// ten strings; each element is "hi!"
forward_list<int> ivec(10);		// ten elements, each initialized to 0
deque<string> svec(10); 		// ten elements, each an empty string
```

#### Library `array`s Have Fixed Size

Just as the size of a built-in array is part of its type, the size of a library `array` is part of its type.**<font color='red'> When we define an `array`, in addition to specifying the element type, we also specify the container size:</font>**

```c++
array<int, 42> 		// type is: array that holds 42 ints
array<string, 10> 	// type is: array that holds 10 strings
    
array<int, 10>::size_type i; 	// array type includes element type and size
array<int>::size_type j; 		// error: array<int> is not a type
```

Unlike the other containers, a default-constructed `array` is not empty: It has as many elements as its size. These elements are default initialized just as are elements in a built-in array.

If we list initialize the `array`, the number of the initializers must be equal to or less than the size of the `array`. If there are fewer initializers than the size of the `array`, the initializers are used for the first elements and any remaining elements are value initialized.

```c++
array<int, 10> ia1; 							// ten default-initialized ints
array<int, 10> ia2 = {0,1,2,3,4,5,6,7,8,9}; 	// list initialization
array<int, 10> ia3 = {42};	 					// ia3[0] is 42, remaining elements are 0
```

**<font color='red'>It is worth noting that although we cannot copy or assign objects of built-in array types, there is no such restriction on `array`</font>**:

```c++
int digs[10] = {0,1,2,3,4,5,6,7,8,9};
int cpy[10] = digs; 				// error: no copy or assignment for built-in arrays
array<int, 10> digits = {0,1,2,3,4,5,6,7,8,9};
array<int, 10> copy = digits; 		// ok: so long as the element type and the size match
```

### 9.2.5. Assignment and `swap`

Container Assignment Operations:

 ![image-20220720111253663](images/image-20220720111253663.png)

If the containers had been of unequal size, **<font color='red'>after the assignment both containers would have the size of the right-hand operand</font>**.

Unlike built-in arrays, the library `array` type does allow assignment. The left- and right-hand operands must have the same type:

```c++
array<int, 10> a1 = {0, 1, 2, 3, 4, 5, 6, 7, 8, 9};
array<int, 10> a2 = {0};		// elements all have value 0
a1 = a2;						// replaces elements in a1
a2 = {0};						// error: cannot assign to an array from a braced list
```

Because the size of the right-hand operand might differ from the size of the left-hand operand, **<font color='red'>the `array` type does not support assign and it does not allow assignment from a braced list of values.</font>**

#### Using `assign` (Sequential Containers Only)

The assignment operator requires that the left-hand and right-hand operands have the same type. 

**<font color='red'>The sequential containers (except `array`) also define a member named `assign` that lets us assign from a different but compatible type, or assign from a
subsequence of a container. </font>** For example, we can use `assign` to assign a range of `char*` values from a `vector` into a list of `string`:

```c++
list<string> names;
vector<const char*> oldstyle;
names = oldstyle; 	// error: container types don't match
// ok: can convert from const char*to string
names.assign(oldstyle.cbegin(), oldstyle.cend());
```

The call to `assign` replaces the elements in `names` with copies of the elements in the range denoted by the iterators.

A second version of `assign` takes an integral value and an element value. It replaces the elements in the container with the specified number of elements, each of which has the specified element value:

```c++
list<string> slist1(1); 		// one element, which is the empty string
slist1.assign(10, "Hiya!"); 	// ten elements; each one is Hiya !
```

#### Using `swap`

The `swap` operation exchanges the contents of two containers of the same type. After the call to `swap`, the elements in the two containers are interchanged.

With the exception of `array`s, swapping two containers is guaranteed to be fast—the elements themselves are not swapped; internal data structures are swapped.

The fact that elements are not moved means that, **<font color='red'>with the exception of `string`, iterators, references, and pointers into the containers are not invalidated.</font>** They refer to the same elements as they did before the `swap`. However, after the `swap`, those elements are in a different container.

In the new library, the containers offer both a member and nonmember version of `swap`. Earlier versions of the library defined only the member version of `swap`. The nonmember `swap` is of most importance in generic programs. As a matter of habit, **<font color='red'>it is best to use the nonmember version of `swap`.</font>**

### 9.2.6. Container Size Operations

 ![image-20220720112945109](images/image-20220720112945109.png)

**<font color='red'>`forward_list` provides `max_size` and `empty`, but not `size`.</font>**

### 9.2.7. Relational Operators

 ![image-20220720113055684](images/image-20220720113055684.png)

The right- and left-hand operands must be the same kind of container and must hold elements of the same type.

#### Relational Operators Use Their Element’s Relational Operator

We can use a relational operator to compare two containers only if the appropriate comparison operator is defined for the element type.

The container equality operators use the element’s `==` operator, and the relational operators use the element’s `<` operator. If the element type doesn’t support the required operator, then we cannot use the corresponding operations on containers holding that type.

## 9.3. Sequential Container Operations

### 9.3.1. Adding Elements to a Sequential Container

 ![image-20220721093829665](images/image-20220721093829665.png)

#### Using `push_back`

Aside from `array` and `forward_list`, every sequential container (including the `string` type) supports `push_back`.

```c++
string word;
while (cin >> word)
	container.push_back(word);
```

The call to `push_back` creates a new element at the end of container, increasing the `size` of `container` by 1. **<font color='red'>The value of that element is a copy of `word`. </font>**The type of `container` can be any of `list`, `vector`, or `deque`.

**<font color='red'>Because `string` is just a container of characters, we can use `push_back` to add characters to the end of the `string`</font>**:

```c++
void pluralize(size_t cnt, string &word){
    if(cnt > 1)
        word.push_back('s');	// same as word += 's'
}
```

> **Key Concept: Container Elements Are Copies**
> When we use an object to initialize a container, or insert an object into a container, a copy of that object’s value is placed in the container, not the object itself. There is no relationship between the element in the container and the object from which that value originated.

#### Using `push_front`

In addition to `push_back`, the `list`, `forward_list`, and `deque` containers support an analogous operation named `push_front`. This operation inserts a new element at the front of the container.

Note that **<font color='red'>`deque`, which like `vector` offers fast random access to its elements, provides the `push_front` member even though `vector` does not.</font>** A `deque` guarantees constant-time insert and delete of elements at the beginning and end of the container. As with `vector`, inserting elements other than at the front or back of a `deque` is a potentially expensive operation.

#### Adding Elements at a Specified Point in the Container

The `insert` members let us insert zero or more elements at any point in the container. The `insert` members are supported for `vector`, `deque`, `list`, and `string`. `forward_list` provides specialized versions of these members that we’ll cover in §9.3.4

Each of the `insert` functions takes an iterator as its first argument. The iterator indicates where in the container to put the element(s). It can refer to any position in the container, including one past the end of the container. **<font color='red'>Element(s) are inserted before the position denoted by the iterator.</font>**

```c++
vector<string> svec;
list<string> slist;
// equivalent to calling slist.push_front("Hello!");
slist.insert(slist.begin(), "Hello!");
// no push_front on vector but we can insert before begin()
// warning: inserting anywhere but at the end of a vector might be slow
svec.insert(svec.begin(), "Hello!");
```

####  Inserting a Range of Elements

The arguments to `insert` that appear after the initial iterator argument are analogous to the container constructors that take the same parameters.

1. The version that takes an element count and a value adds the specified number of identical elements before the given position:

   ```c++
   svec.insert(svec.end(), 10, "Anna")
   ```

   This code inserts ten elements at the end of `svec` and initializes each of those elements to the `string "Anna"`.

2. The versions of insert that take a pair of iterators or an initializer list insert the elements from the given range before the given position

   ```c++
   vector<string> v = {"quasi", "simba", "frollo", "scar"};
   // insert the last two elements of v at the beginning of slist
   slist.insert(slist.begin(), v.end() - 2, v.end());
   slist.insert(slist.end(), {"these", "words", "will", "go", "at", "the", "end"});
   ```

   When we pass a pair of iterators,**<font color='red'> those iterators may not refer to the same container as the one to which we are adding elements.</font>**

   ```c++
   // run-time error: iterators denoting the range to copy from
   // must not refer to the same container as the one we are changing
   slist.insert(slist.begin(), slist.begin(), slist.end());
   ```

#### ⭐Using the Return from insert

The versions of `insert` that take a count or a range return an iterator to the first element that was inserted. If the range is empty, no elements are
inserted, and the operation returns its first parameter.

We can use the value returned by `insert` to repeatedly insert elements at a specified position in the container:

```c++
list<string> lst;
auto iter = lst.begin();
while(cin >> word)
    iter = lst.insert(iter, word);	// same as calling push_front
```

#### Using the Emplace Operations

The new standard introduced three new members—`emplace_front`, `emplace`, and `emplace_back`—that **<font color='red'>construct rather than copy elements</font>**. 

When we call a `push` or `insert` member, we pass objects of the element type and those objects are copied into the container. When we call an `emplace` member, we pass arguments to a constructor for the element type. **<font color='red'>The `emplace` members use those arguments to construct an element directly in space managed by the container.</font>**

```c++
// uses the three-argument Sales_data constructor
c.emplace_back("978-0590353403", 25, 15.99);
// error: there is no version of push_back that takes three arguments
c.push_back("978-0590353403", 25, 15.99);
// ok: we create a temporary Sales_data object to pass to push_back
c.push_back(Sales_data("978-0590353403", 25, 15.99));
```

In the call to `emplace_back`, that object is created directly in space managed by the container. The call to `push_back` creates a local temporary object that is pushed onto the container.

The arguments to an `emplace` function vary depending on the element type. The arguments must match a constructor for the element type:

```c++
// iter refers to an element in c, which holds Sales_data elements
c.emplace_back(); 					// uses the Sales_data default constructor
c.emplace(iter, "999-999999999"); 	// uses Sales_data(string)
// uses the Sales_data constructor that takes an ISBN, a count, and a price
c.emplace_front("978-0590353403", 25, 15.99);
```

### 9.3.2. Accessing Elements

 ![image-20220721102106189](images/image-20220721102106189.png)

Each sequential container, including `array`, has a `front` member, and all except `forward_list` also have a back member. These operations return a reference to the first and last element, respectively.

#### The Access Members Return References

The members that access elements in a container (i.e., `front`, `back`, `subscript`, and `at`) return references. If the container is a `const` object, the return is a reference to `const`. If the container is not `const`, the return is an ordinary reference that we can use to change the value of the fetched element:

```c++
if(!c.empty()){
    c.front() = 42;			// assigns 42 to the first element in c
    auto &v = c.back();		// get a reference to the last element
    v = 1024;				// changes the element in c
    auto v2 = c.back();		// v2 is not a reference; it's a copy of c.back()
    v2 = 0;					// no change to the element in c
}
```

#### Subscripting and Safe Random Access

The containers that provide fast random access (`string`, `vector`, `deque`, and `array`) also provide the subscript operator. The subscript operator does not check whether the index is in range. Using an out-of-range value for an index is a serious programming error, but one that the compiler will not detect.

If we want to ensure that our index is valid, we can use the `at` member instead. **<font color='red'>The `at` member acts like the subscript operator, but if the index is invalid, `at` throws an `out_of_range` exception.</font>**

```c++
vector<string> svec; 	// empty vector
cout << svec[0]; 		// run-time error: there are no elements in svec!
cout << svec.at(0); 	// throws an out_of_range exception
```

### 9.3.3. Erasing Elements

 ![image-20220721103548800](images/image-20220721103548800.png)

#### The `pop_front` and `pop_back` Members

The `pop_front` and `pop_back` functions remove the first and last elements, respectively. Just as there is no `push_front` for `vector` and `string`, there is also no `pop_front` for those types. Similarly, `forward_list` does not have pop_back.

These operations return `void`. If you need the value you are about to pop, you must store that value before doing the pop:

```c++
while (!ilist.empty()) {
	process(ilist.front()); 	// do something with the current top of ilist
	ilist.pop_front(); 			// done; remove the first element
}
```

#### ⭐Removing an Element from within the Container

The `erase` members remove element(s) at a specified point in the container. We can delete a single element denoted by an iterator or a range of elements marked by a pair of iterators. **<font color='red'>Both forms of `erase` return an iterator referring to the location after the (last) element that was removed.</font>**

```c++
// erases the odd elements in a list
list<int> lst = {0,1,2,3,4,5,6,7,8,9};
auto it = lst.begin();
while(it != lst.end()){
    if(*it % 2){				// if the element is odd
        it = list.erase(it);	// erase this element
    }
    else{
        ++it;
    }
}
```

#### Removing Multiple Elements

The iterator-pair version of `erase` lets us delete a range of elements:

```c++
elem1 = slist.erase(elem1, elem2);	// after the call elem1 = elem2
```

The iterator `elem1` refers to the first element we want to erase, and `elem2` refers to **<font color='red'>one past the last</font>** element we want to remove. 

To delete all the elements in a container, we can either call `clear` or pass the iterators from `begin` and `end` to `erase`:

```c++
slist.clear();								// delete all the elements within the container
slist.erase(slist.begin(), slist.end());	// equivalent
```

### 9.3.4. Specialized `forward_list` Operations

Consider what must happen when we remove an element from a singly linked list:

 ![image-20220721104759646](images/image-20220721104759646.png)

Removing `elem3` changes `elem2`; `elem2` had pointed to `elem3`, but after we remove `elem3`, `elem2` points to `elem4`.

When we add or remove an element, the element before the one we added or removed has a different successor. To add or remove an element, we need access to its predecessor in order to update that element’s links. However, `forward_list` is a singly linked list. In a singly linked list there is no easy way to get to an element’s predecessor. For this reason, **<font color='red'>the operations to add or remove elements in a `forward_list` operate by changing the element after the given element.</font>**

For example, in our illustration, to remove `elem3`, we’d call `erase_after` on an iterator that denoted `elem2`. To support these operations, **<font color='red'>`forward_list` also defines `before_begin`, which returns an off-the-beginning iterator.</font>** This iterator lets us add or remove elements “after” the nonexistent element before the first one in the list.

 ![image-20220721105217461](images/image-20220721105217461.png)

**<font color='red'>When we add or remove elements in a `forward_list`, we have to keep track of two iterators—one to the element we’re checking and one to that element’s predecessor.</font>** As an example, we’ll rewrite the loop that removed the odd-valued elements from a `list` to use a `forward_list`:

```c++
// erases the odd elements in a forward_list
forward_list<int> flst = {0,1,2,3,4,5,6,7,8,9};
auto prev = flst.before_begin();
auto curr = flst.begin();
while(curr != flst.end()){
    if(*curr % 2){		// if the element is odd
        curr = flst.erase_after(prev);
    }
    else{
        prev = curr;
        ++curr;
    }
}
```

When we find an odd element, we pass `prev` to `erase_after`. This call erases the element after the one denoted by `prev`; that is, it erases the element denoted by `curr`. We reset `curr` to the return from `erase_after`, which makes `curr` denote the next element in the sequence and we leave `prev` unchanged; `prev` still denotes the element before the (new) value of `curr`. 

If the element denoted by `curr` is not odd, then we have to move both iterators, which we do in the `else`.

### 9.3.5. Resizing a Container

 ![image-20220721112507111](images/image-20220721112507111.png)

If the current size is greater than the requested size, elements are deleted from the back of the container; if the current size is less than the new size, elements are added to the back of the container:

```c++
list<int> ilist(10, 42); 		// ten ints: each has value 42
ilist.resize(15); 				// adds five elements of value 0 to the back of ilist
ilist.resize(25, -1); 			// adds ten elements of value -1 to the back of ilist
ilist.resize(5); 				// erases 20 elements from the back of ilist
```

The `resize` operation takes an optional element-value argument that it uses to initialize any elements that are added to the container. If this argument is absent, added elements are **<font color='red'>value initialized</font>**.

### 9.3.6. Container Operations May Invalidate Iterators

Operations that add or remove elements from a container can invalidate pointers, references, or iterators to container elements. It is a serious run-time error to use an iterator, pointer, or reference that has been invalidated.

When you use an iterator (or a reference or pointer to a container element), it is a good idea to minimize the part of the program during which an iterator must stay valid. Because code that adds or removes elements to a container can invalidate iterators, **<font color='red'>you need to ensure that the iterator is repositioned</font>**, as appropriate, **<font color='red'>after each operation that changes the container</font>**. This advice is especially important for `vector`, `string`, and `deque`.

#### Writing Loops That Change a Container

Loops that add or remove elements of a `vector`, `string`, or `deque` must cater to the fact that iterators, references, or pointers might be invalidated. The program must ensure that the iterator, reference, or pointer**<font color='red'> is refreshed on each trip</font>** through the loop. Refreshing an iterator is easy if the loop calls `insert` or `erase`. Those operations return iterators, which we can use to reset the iterator:

```c++
// remove even-valued elements and insert a duplicate of odd-valued elements
vector<int> vi = {0,1,2,3,4,5,6,7,8,9};
auto iter = vi.begin();
while(iter != vi.end()){
    if(*iter % 2){
        iter = vi.insert(iter, *iter);	// duplicate the current element
        iter += 2;
    }
    else{
        iter = vi.erase(iter);			// remove even elements
    }
}
```

We refresh the iterator after both the `insert` and the `erase` because either operation can invalidate the iterator.

* After the call to `erase`, there is no need to increment the iterator, because the iterator returned from `erase` denotes the next element in the sequence.
* After the call to `insert`, we increment the iterator twice. Remember, `insert` inserts before the position it is given and returns an iterator to the inserted element. Thus, after calling `insert`, `iter` denotes the (newly added) element in front of the one we are processing. We add two to skip over the element we added and the one we just processed. Doing so positions the iterator on the next, unprocessed element.

#### Avoid Storing the Iterator Returned from end

When we add or remove elements in a `vector` or `string`, or add elements or remove any but the first element in a `deque`, the iterator returned by `end` is always invalidated. Thus, **<font color='red'>loops that add or remove elements should always call `end` rather than use a stored copy.</font>**

```c++
// disaster: the behavior of this loop is undefined
auto begin = v.begin(),
end = v.end(); 			// bad idea, saving the value of the end iterator
while (begin != end) {
	// do some processing
	++begin;
	begin = v.insert(begin, 42);
	++begin;
}
```

The behavior of this code is undefined. On many implementations, we’ll get an infinite loop. The problem is that we stored the value returned by the `end` operation in a local variable named `end`. In the body of the loop, we added an element. Adding an element invalidates the iterator stored in `end`. That iterator neither refers to an element in `v` nor any longer refers to one past the last element in `v`.

## 9.4. How a `vector` Grows

To support fast random access, `vector` elements are stored contiguously—each element is adjacent to the previous element. 

When they have to get new memory, `vector` and `string` implementations typically allocate capacity beyond what is immediately needed. The container holds this storage in reserve and uses it to allocate new elements as they are added. Thus, there is no need to reallocate the container for each new element.

#### Members to Manage Capacity

 ![image-20220721115200940](images/image-20220721115200940.png)

The `vector` and `string` types provide members that let us interact with the memory-allocation part of the implementation. 

* The `capacity` operation tells us how many elements the container can hold before it must allocate more space. 

* The `reserve` operation lets us tell the container how many elements it should be prepared to hold. 

  A call to `reserve` changes the capacity of the `vector` only if the requested space exceeds the current capacity. If the requested size is greater than the current capacity, `reserve` allocates at least as much as (and may allocate more than) the requested amount.**<font color='red'> If the requested size is less than or equal to the existing capacity, `reserve` does nothing.</font>**

  **<font color='red'>As a result, a call to `reserve` will never reduce the amount of space that the container uses.</font>**

* Under the new library, we can call `shrink_to_fit` to ask a `deque`, `vector`, or `string` to return unneeded memory. This function indicates that we no longer need any excess capacity.

#### `capacity` and `size`

The `size` of a container is the number of elements it already holds; its `capacity` is how many elements it can hold before more space must be allocated.

 ![image-20220721115920417](images/image-20220721115920417.png)

A `vector` may be reallocated **<font color='red'>only </font>**when the user performs an insert operation when the `size` equals `capacity` or by a call to `resize` or `reserve` with a value that exceeds the current `capacity`.
