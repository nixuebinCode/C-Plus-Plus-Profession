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
