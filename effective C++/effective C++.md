

# 1. Accustoming Yourself to C++

## Item 1: View C++ as a federation of languages

C++ is a multiparadigm programming language. To make sense of C++, you have to recognize its primary sublanguages:

* **C**

  Way down deep, C++ is still based on C. Blocks, statements, the preprocessor, built-in data types, arrays, pointers, etc., all come from C.

* **Object-Oriented C++**

  Classes (including constructors and destructors), encapsulation, inheritance, polymorphism, virtual functions (dynamic binding), etc.

* **Template C++**

  The generic programming part of C++

* **The STL**

  The STL is a template library with containers, iterators, algorithms, and function objects.

C++, isn't a unified language with a single set of rules; it's a federation of four sublanguages, each with its own conventions.

## Item 2: Prefer consts, enums, and inlines to #defines

**Prefer the compiler to the preprocessor.**

When you do something like this:

```c++
#define ASPECT_RATIO 1.653
```

The symbolic name ASPECT_RATIO may never be seen by compilers; it may be removed by the preprocessor before the source code ever gets to a compiler. As a result, This can be confusing if you get an error during compilation involving the use of the constant, because the error message may refer to 1.653, not ASPECT_RATIO.

The solution is to replace the macro with a constant:

```c++
const double AspectRatio = 1.653;
```

In the case of a floating point constant (such as in this example), use of the constant may yield smaller code than using a #define:

That's because the preprocessor's blind substitution of the macro name ASPECT_RATIO with 1.653 could result in multiple copies of 1.653 in your object code, while the use of the constant AspectRatio should never result in more than one copy.

### class-specific constants

To limit the scope of a constant to a class, you must make it a member, and to ensure there's at most one copy of the constant, you must make it a static member:

```c++
class GamePlayer {
private:
	static const int NumTurns = 5; // constant declaration
	int scores[NumTurns]; // use of constant
	...
};
```

**What you see above is a declaration for NumTurns, not a definition.**

Usually, C++ requires that you provide a definition for anything you use, but class-specific constants that are static and of integral type (e.g., integers, chars, bools) are an exception.

As long as you don't take their address, you can declare them and use them without providing a definition.

If you do take the address of a class constant, or if your compiler incorrectly insists on a definition even if you don't take the address, you provide a **separate definition** like this:

```c++
const int GamePlayer::NumTurns; // definition of NumTurns;
```

Because the initial value of class constants is provided where the constant is declared (e.g., NumTurns is initialized to 5 when it is declared), no initial value is permitted at the point of definition.

Older compilers may not accept the syntax above, because it used to be illegal to provide an initial value for a static class member at its point of declaration. Instead, you can put the initial value at the point of definition:

```c++
class CostEstimate {
private:
	static const double FudgeFactor; 	// declaration of static class
	... 									// constant; goes in header file
};
const double CostEstimate::FudgeFactor = 1.35; // definition of static class constant; goes in impl. file
```

This is all you need almost all the time. The only exception is when you need the value of a class constant during compilation of the class, such as in the
declaration of the array GamePlayer::scores above. Then the accepted way to compensate for compilers that (incorrectly) forbid the in-class specification
of initial values for static integral class constants is to use enum.

```c++
class GamePlayer {
private:
	enum { NumTurns = 5 }; // "the enum hack" — makes NumTurns a symbolic name for 5
	int scores[NumTurns]; // fine
	...
};
```

This technique takes advantage of the fact that the values of an enumerated type can be used where ints are expected

### function-like macros

Using #define to implement macros that look like functions but that don't incur the overhead of a function call:

```c++
// call f with the maximum of a and b
#define CALL_WITH_MAX(a, b) f((a) > (b) ? (a) : (b))
```

Macros like this have so many drawbacks:

Whenever you write this kind of macro, you have to remember to parenthesize all the arguments in the macro body. Otherwise you can run into
trouble when somebody **calls the macro with an expression**. But even if you get that right, look at the weird things that can happen:

```c++
int a = 5, b = 0;
CALL_WITH_MAX(++a, b); // a is incremented twice
CALL_WITH_MAX(++a, b+10); // a is incremented once
```

You can get all the efficiency of a macro plus all the predictable behavior and type safety of a regular function by using **a template for an inline function**:

```c++
template<typename T>
inline void callWithMax(const T& a, const T& b)
{ 
	f(a > b ? a : b); 
}
```



## Item3: Use const whenever possible

### const for Pointers

```c++
char greeting[] = "Hello";
char *p = greeting; 				// non-const pointer,
									// non-const data
const char *p = greeting; 			// non-const pointer,
									// const data
char * const p = greeting; 			// const pointer,
									// non-const data
const char * const p = greeting; 	// const pointer,
									// const data
```

### const for Iterators

An iterator acts much like a T* pointer. Declaring an iterator const is like declaring a pointer const (i.e., declaring a T* const pointer)

```c++
std::vector<int> vec;
const std::vector<int>::iterator iter = vec.begin();	// iter acts like a T* const
*iter = 10; 	// OK, changes what iter points to
++iter; 		// error! iter is const

std::vector<int>::const_iterator cIter = vec.begin();	//cIter acts like a const T*
*cIter = 10; 	// error! *cIter is const
++cIter; 		// fine, changes
```

### const Member Functions

The purpose of const on member functions is to identify which member functions may be invoked on const objects.

One of the fundamental ways to improve a C++ program's performance is to pass objects by reference-to-const. That technique is viable only if there are const member functions with which to manipulate the resulting const-qualified objects.

#### member functions differing only in their constness can be overloaded

```c++
class TextBlock {
public:
	...
	const char& operator[](std::size_t position) const //operator[] for const objects
		{ return text[position]; }
	char& operator[](std::size_t position) //operator[] for non-const objects
		{ return text[position]; } //
private:
	std::string text;
};
```

```c++
TextBlock tb("Hello");
std::cout << tb[0]; // calls non-const TextBlock::operator[]

const TextBlock ctb("World");
std::cout << ctb[0]; // calls const TextBlock::operator[]
```

#### What does it mean for a member function to be const?
* bitwise constness

  A member function is const if and only if it doesn't modify any of the object's data members (excluding those that are static).

  Bitwise constness is C++'s definition of constness, and a const member function isn't allowed to modify any of the non-static data members of the object on which it is invoked.

  Unfortunately, many member functions that don't act very const pass the bitwise test.

  ```c++
  class CTextBlock {
  public:
  	...
  	char& operator[](std::size_t position) const 	// inappropriate (but bitwise const) declaration of 													//operator[]
  		{ return pText[position]; }
  private:
  	char *pText;
  };
  ```

  operator[]'s implementation doesn't modify pText in any way. As a result, compilers will happily generate code for operator[]; it is, after all, bitwise const, and that's all compilers check for.

  ```c++
  const CTextBlock cctb("Hello"); // declare constant object
  char *pc = &cctb[0]; // call the const operator[] to get a pointer to cctb's data
  *pc = 'J'; // cctb now has the value "Jello"
  ```

* logical constness

  A const member function might modify some of the bits in the object on which it's invoked, but only in ways that clients cannot detect.

  Take advantage of C++'s const-related wiggle room known as mutable.

  mutable frees non-static data members from the constraints of bitwise constness.

  ```c++
  class CTextBlock {
  public:
  	...
  	std::size_t length() const;
  private:
  	char *pText;
  	mutable std::size_t textLength; // these data members may always be modified, even in const member 										// functions
  	mutable bool lengthIsValid;
  };
  
  std::size_t CTextBlock::length() const
  {
  	if (!lengthIsValid) {
  		textLength = std::strlen(pText); // now fine
  		lengthIsValid = true; // also fine
  	}
  	return textLength;
  }
  ```

**Compilers enforce bitwise constness, but you should program using logical constness.**

### Avoiding Duplication in const and Non-const Member Functions

Sometimes, there may be duplicate processes in both const and non-const member functions.  In this case, the const version of operator[] does exactly what the non-const version does, it just has a const-qualified return type. So having the non-const operator[] call the const version is a safe way to avoid code duplication, even though it requires a cast.

```c++
class TextBlock {
public:
	...
	const char& operator[](std::size_t position) const // same as before
	{
		...
		...
		...
		return text[position];
	}
	char& operator[](std::size_t position) // now just calls const op[]
	{
		return const_cast<char&>(static_cast<const TextBlock&>(*this)[position]); 	// cast away const on op[]'s return type;
 																					// add const to *this's type;
                                 													// call const version of op[]

    }
	...
};
```

We have two casts

* one to add const to *this (so that our call to operator[] will call the const version)
* the second to remove the const from the const operator[]'s return value

The cast that adds const is just forcing a safe conversion (from a non-const object to a const one), so we use a static_cast for that.

The one that removes const can be accomplished only via a const_cast, so we don't really have a choice there.

Having a const member function call a non-const one is wrong: A const member function promises never to change the logical state of its object, but a non-const member function makes no such promise. If you were to call a non-const function from a const one, you'd run the risk that the object you'd promised not to modify would be changed.

**When const and non-const member functions have essentially identical implementations, code duplication can be avoided by having the non-const version call the const version.**

## Item4: Make sure that objects are initialized before they're used

### non-member objects of built-in types

For non-member objects of built-in types, you'll need to initialize your objects manually.

```c++
int x = 0; 									// manual initialization of an int
const char * text = "A C-style string"; 	// manual initialization of a pointer
double d; 									// "initialization" by reading from
std::cin >> d; 								// an input stream
```

### member objects

For almost everything else, the responsibility for initialization falls on constructors. The rule there is simple: make sure that all constructors initialize everything in the object. But it's important not to confuse assignment with initialization.

```c++
ABEntry::ABEntry(const std::string& name, const std::string& address, const std::list<PhoneNumber>& phones)
{
	theName = name; 		// these are all
							// assignments,
	theAddress = address; 	// not initializations
	thePhones = phones;
	numTimesConsulted = 0;
}
```

Inside the ABEntry constructor, theName, theAddress, and thePhones aren't being initialized, they're being assigned. Initialization took place earlier — when their default constructors were automatically called prior to entering the body of the ABEntry constructor.

This isn't true for numTimesConsulted, because it's a built-in type. For it, there's no guarantee it was initialized at all prior to its assignment.

We should use the member initialization list:

```c++
ABEntry::ABEntry(const std::string& name, const std::string& address, const std::list<PhoneNumber>& phones)
					: theName(name),
					theAddress(address), 	// these are now all
											//initializations
					thePhones(phones),
					numTimesConsulted(0)
{} // the ctor body is now empty
```

* The assignment-based version first called default constructors to initialize theName, theAddress, and thePhones, then promptly assigned new values on top of the default-constructed ones. All the work performed in those default constructions was therefore wasted.
* The member initialization list approach avoids that problem, because the arguments in the initialization list are used as constructor arguments for the various data members.
* Many classes have multiple constructors, and each constructor has its own member initialization list. If there are many data members and/or base classes, the existence of multiple initialization lists introduces undesirable repetition (in the lists) and boredom (in the programmers). In such cases, moving the assignments to a single (typically private) function that all the constructors call may be helpful.
* **Within a class, data members are initialized in the order in which they are declared regardless of the order in the member initialization list.**

### non-local static objects defined in different translation units

A static object is one that exists from the time it's constructed until the end of the program.

* global objects
* objects defined at namespace scope
* objects declared static inside classes
* objects declared static inside functions
* objects declared static at file scope

Static objects inside functions are known as local static objects (because they're local to a function), and the other kinds of static objects are known as non-local static objects.

A translation unit(编译单元) is the source code giving rise to a single object file(单一目标文件). It's basically a single source file, plus all of its #include files.

**The relative order of initialization of non-local static objects defined in different translation units is undefined.**

```c++
class FileSystem { // from your library’s header file
public:
	...
	std::size_t numDisks() const; // one of many member functions
	...
};
extern FileSystem tfs; 	// declare object for clients to use
						// ("tfs" = "the filesystem"); definition is in some .cpp file in your library
```

```c++
class Directory { // created by library client
public:
	Directory( params );
	...
};
Directory::Directory( params )
{
	...
	std::size_t disks = tfs.numDisks(); // use the tfs object
	...
}
Directory tempDir( params ); // directory for temporary files
```

Now the importance of initialization order becomes apparent: unless tfs is initialized before tempDir, tempDir's constructor will attempt to use tfs before it's been initialized.

**Solution: move each non-local static object into its own function, where it's declared static. These functions return references to the objects they contain. Clients then call the functions instead of referring to the objects.**

This approach is founded on C++'s guarantee that local static objects are initialized when the object's definition is first encountered during a call to that function.

```c++
class FileSystem { ... }; 	// as before
FileSystem& tfs() 			// this replaces the tfs object; it could be
{ 							// static in the FileSystem class
	static FileSystem fs; 	// define and initialize a local static object
	return fs; 				// return a reference to it
}
class Directory { ... }; 			// as before
Directory::Directory( params ) 		// as before, except references to tfs are
{ 									// now to tfs()
	...
	std::size_t disks = tfs().numDisks();
	...
}
Directory& tempDir() 					// this replaces the tempDir object; it
{ 										// could be static in the Directory class
	static Directory td; ( params ); 	// define/initialize local static object
	return td;							// return reference to it
}
```

***

# 2. Constructors, Destructors, and sAssignment Operators
## Item 5: Know what functions C++ silently writes and calls

If you don't declare them yourself, compilers will declare their own versions of a copy constructor, a copy assignment operator, and a destructor.
Furthermore, if you declare no constructors at all, compilers will also declare a default constructor for you. All these functions will be both <font color=#FF0000>public and
inline</font>.

### default constructor and destructor

The default constructor and the destructor primarily give compilers a place to put “behind the scenes” code such as invocation of constructors and destructors of <font color=#FF0000>base classes and non-static data members</font>.

### default copy constructor and the copy assignment operator

The compiler generated versions simply copy each <font color=#FF0000>non-static data member</font> of the source object over to the target object.

```c++
template<typename T>
class NamedObject {
public:
	NamedObject(const char *name, const T& value);
	NamedObject(const std::string& name, const T& value);
	...
private:
	std::string nameValue;
	T objectValue;
};
```

```c++
NamedObject<int> no1("Smallest Prime Number", 2);
NamedObject<int> no2(no1); // calls copy constructor
```

The copy constructor generated by compilers must initialize no2.nameValue and no2.objectValue using no1.nameValue and no1.objectValue:

* The type of nameValue is string, and the standard string type has a copy constructor, so no2.nameValue will be initialized by <font color=#FF0000>calling the string copy constructor</font> with no1.nameValue as its argument.
* The type of objectValue is int , and int is a built-in type, so no2.objectValue will be initialized by copying the bits in no1.objectValue

### Compilers  may refuse to generate an operator= for class

1. Classes containing reference members

   ```c++
   template<typename T>
   class NamedObject {
   public:
   	// this ctor no longer takes a const name, because nameValue
   	// is now a reference-to-non-const string. The char* constructor
   	// is gone, because we must have a string to refer to.
   	NamedObject(std::string& name, const T& value);
   	... // as above, assume no
   		// operator= is declared
   private:
   	std::string& nameValue; // this is now a reference
   	const T objectValue; // this is now const
   };
   ```

   ```c++
   std::string newDog("Persephone");
   std::string oldDog("Satch");
   NamedObject<int> p(newDog, 2);
   NamedObject<int> s(oldDog, 36);
   p = s; // what should happen to the data members in p?
   ```

   How should the assignment affect p.nameValue? 

   * Should p.nameValue refer to the string referred to by s.nameValue, i.e., should the reference itself be modified?

     C++ doesn't provide a way to make a reference refer to a different object.

   * Should the string object to which p.nameValue refers be modified, thus affecting other objects that hold pointers or
     references to that string.

   Faced with this conundrum, C++ refuses to compile the code.

2. Classes containing const members

   Compilers behave similarly for classes containing const members. It's not legal to modify const members, so compilers are unsure how
   to treat them during an implicitly generated assignment function.

3. Derived classes that inherit from base classes declaring the copy assignment operator private.

   Compiler-generated copy assignment operators for derived classes are supposed to handle base class parts, too, but in doing so,
   they certainly can't invoke member functions the derived class has no right to call.

***

## Item 6: Explicitly disallow the use of compiler generated functions you do not want
### Declaring member functions private and deliberately not implementing them

By declaring a member function explicitly, you prevent compilers from generating their own version.

By making the function private, you keep people from calling it.

But member and friend functions can still call your private functions. By deliberately not implementing them, if somebody inadvertently calls one, they'll get an error at link-time.

```c++
class HomeForSale {
public:
	...
private:
	...
	HomeForSale(const HomeForSale&); // declarations only
	HomeForSale& operator=(const HomeForSale&);
};
```

### Declaring the copy constructor and copy assignment operator private in a base class specifically designed to prevent copying

It's possible to move the link-time error up to compile time (always a good thing — earlier error detection is better than later).

```c++
class Uncopyable {
protected: 				// allow construction
	Uncopyable() {} 	// and destruction of
	~Uncopyable() {} 	// derived objects...
private:
	Uncopyable(const Uncopyable&); // ...but prevent copying
	Uncopyable& operator=(const Uncopyable&);
};
```

To keep HomeForSale objects from being copied, all we have to do now is inherit from Uncopyable

```c++
class HomeForSale: private Uncopyable { // class no longer
	...									// declares copy ctor or
}; 										// copy assign operator
```

The compiler-generated versions of these functions will try to call their base class counterparts, and those calls will be rejected, because the copying operations are private in the base class.

### Defining a Function as Deleted

Under the new standard, we can prevent copies by defining the copy constructor and copy-assignment operator as deleted functions.

A deleted function is one that is declared but may not be used in any other way. We indicate that we want to define a function as deleted by following its parameter list with = delete.

```c++
struct NoCopy {
	NoCopy() = default; 						// use the synthesized default constructor
	NoCopy(const NoCopy&) = delete; 			// no copy
	NoCopy &operator=(const NoCopy&) = delete; 	// no assignment
    ~NoCopy() = default; 						// use the synthesized destructor
	// other members
};
```

***

## Item 7: Declare destructors virtual in polymorphic base classes

### Virtual destructors in polymorphic base classes

Suppose we create a TimeKeeper base class along with derived classes for different approaches to timekeeping.

And we have a factory function — a function that returns a base class pointer to a newly-created derived class object — can be used to return
a pointer to a timekeeping object

```c++
class TimeKeeper {
public:
	TimeKeeper();
	~TimeKeeper();
	...
};
class AtomicClock: public TimeKeeper { ... };
class WaterClock: public TimeKeeper { ... };
class WristWatch: public TimeKeeper { ... };

TimeKeeper* getTimeKeeper(); 	// returns a pointer to a dynamically
								// allocated object of a class derived from TimeKeeper
```

The problem is that getTimeKeeper returns a pointer to a derived class object (e.g., AtomicClock), that object is being deleted via a base class pointer (i.e., a TimeKeeper* pointer), and the base class (TimeKeeper) has a non-virtual destructor.

C++ specifies that when a derived class object is deleted through a pointer to a base class with a nonvirtual destructor, results are undefined:
**The derived part of the object is never destroyed.** 
**However, the base class part (i.e., the TimeKeeper part) typically would be destroyed, thus leading to a curious “partially destroyed” object.**
**This is an excellent way to leak resources, corrupt data structures, and spend a lot of time with a debugger.**

### In some cases, making the destructor virtual may be a bad idea

When a class is not intended to be a base class, making the destructor virtual is usually a bad idea.

```c++
class Point { // a 2D point
public:
	Point(int xCoord, int yCoord);
	~Point();
private:
	int x, y;
};
```

The implementation of virtual functions requires that objects carry information that can be used at runtime to determine which virtual functions should be invoked on the object.

This information typically takes the form of a pointer called a vptr (“virtual table pointer”). The vptr points to an array of function pointers called a vtbl (“virtual table”).

When a virtual function is invoked on an object, the actual function called is determined by following the object's vptr to a vtbl and then looking up the appropriate function pointer in the vtbl.

**If the Point class contains a virtual function, objects of that type will increase in size. Addition of a vptr to Point will thus increase its size by 50–100%!**

The standard string type and STL container types contain no virtual functions, so we should never inherit from those classes.

### Pure virtual destructor

Recall that pure virtual functions result in abstract classes — classes that can't be instantiated.

Sometimes, however, you have a class that you'd like to be abstract, but you don't have any pure virtual functions. The solution is simple: declare a pure virtual destructor in the class you want to be abstract.

```c++
class AWOV {
public:
	virtual ~AWOV() = 0; // declare pure virtual destructor
};
```

**There is one twist, however: you must provide a definition for the pure virtual destructor:**

```c++
AWOV::~AWOV() {} // definition of pure virtual dtor
```

*The way destructors work is that the most derived class's destructor is called first, then the destructor of each base class is called. Compilers will generate a call to ~AWOV from its derived classes' destructors, so you have to be sure to provide a body for the function. If you don't, the linker will complain.*

***

## Item 8: Prevent exceptions from leaving destructors

Premature program termination or undefined behavior can result from destructors emitting exceptions.

C++ does not like destructors that emit exceptions!

```c++
class Widget {
public:
	...
	~Widget() { ... } // assume this might emit an exception
};
void doSomething()
{
	std::vector<Widget> v;
	...
} // v is automatically destroyed here
```

Suppose v has ten Widgets in it, and during destruction of the first one, an exception is thrown. The other nine Widgets still have to be destroyed (otherwise any resources they hold would be leaked), so v should invoke their destructors. But suppose that during those calls, a second Widget destructor throws an exception. Now there are two simultaneously active exceptions, and that's one too many for C++.

Depending on the precise conditions under which such pairs of simultaneously active exceptions arise, program execution either terminates or yields undefined behavior.

### What should you do if your destructor needs to perform an operation that may fail by throwing an exception?

Suppose you're working with a class for database connections. And you have a resource-managing class for DBConnection that calls close in its desturctor.

```c++
class DBConnection {
public:
	...
	static DBConnection create(); // function to return DBConnection objects
	void close(); 	// close connection;
					//throw an exception if closing fails
};

class DBConn { 	// class to manage DBConnection objects
public:
	...
	~DBConn() // make sure database connections are always closed
	{
		db.close();
	}
private:
	DBConnection db;
};
```

This is fine as long as the call to close succeeds, but if the call yields an exception, DBConn's destructor will propagate that exception. That's a problem, because destructors that throw mean trouble.

#### Solution 1

Terminate the program if close throws, typically by calling `abort`

```c++
DBConn::~DBConn()
{ 
	try {
		db.close();
    }
	catch (...) {
		make log entry that the call to close failed;
		std::abort();
	}
}
```

Calling abort may forestall undefined behavior.

#### Solution 2

Swallow the exception arising from the call to close.

```c++
DBConn::~DBConn()
{
    try {
		db.close();
    }
	catch (...) {
		make log entry that the call to close failed;
	}
}
```

In general, swallowing exceptions is a bad idea, because it suppresses important information — something failed!

For this to be a viable option, the program must be able to reliably continue execution even after an error has been encountered and ignored.

#### Solution 3

Neither of approaches above is especially appealing. The problem with both is that the program has no way to react to the condition that led to close
throwing an exception in the first place.

A better strategy is to design DBConn's interface so that its clients have an opportunity to react to problems that may arise.

```c++
class DBConn {
public:
	...
	void close() // new function for client use
	{
        db.close();
		closed = true;
	}
	~DBConn()
	{
    	if (!closed) {	// close the connection if the client didn't
			try { 
				db.close();
			}
			catch (...) { // if closing fails, note that and terminate or swallow
				make log entry that call to close failed;
				...
			}
		}
	}
private:
	DBConnection db;
	bool closed;
};
```

The rule is that:

If an operation may fail by throwing an exception and there may be a need to handle that exception, the exception has to come from some non-destructor function.

In this example, telling clients to call close themselves doesn't impose a burden on them; it gives them an opportunity to deal with errors they would otherwise have no chance to react to.

If they don't find that opportunity useful (perhaps because they believe that no error will really occur), they can ignore it, relying on DBConn's destructor to call close for them. If an error occurs at that point — if close does throw — they're in no position to complain if DBConn swallows the exception or terminates the program.

## Item 9: Never call virtual functions during construction or destruction

Suppose you've got a class hierarchy for modeling stock transactions:

```c++
class Transaction { // base class for all transactions
public:
	Transaction();
	virtual void logTransaction() const = 0; // make type-dependent log entry
	...
};
Transaction::Transaction() // implementation of base class ctor
{
	...
	logTransaction(); // as final action, log this transaction
}

class BuyTransaction: public Transaction { // derived class
public:
	virtual void logTransaction() const; // how to log transactions of this type
	...
};

class SellTransaction: public Transaction { // derived class
public:
	virtual void logTransaction() const; // how to log transactions of this type
	...
};


```

When the code `BuyTransaction b;` is executed, a BuyTransaction constructor will be called, but first, a Transaction constructor must be called.
Base class parts of derived class objects are constructed before derived class parts are. The last line of the Transaction constructor calls the virtual function logTransaction. 
The version of logTransaction that's called is the one in Transaction, not the one in BuyTransaction — even though the type of object being created is BuyTransaction.

There's a good reason for this seemingly counterintuitive behavior. Because base class constructors execute before derived class constructors, derived
class data members have not been initialized when base class constructors run. If virtual functions called during base class construction went down to
derived classes, the derived class functions would almost certainly refer to local data members, but those data members would not yet have been initialized.

The same reasoning applies during destruction. Once a derived class destructor has run, the object's derived class data members assume undefined
values, so C++ treats them as if they no longer exist. Upon entry to the base class destructor, the object becomes a base class.

### how to ensure that the proper version of logTransaction is called each time an object in the Transaction hierarchy is created

Turn logTransaction into a non-virtual function in Transaction, then require that derived class constructors pass the necessary log information to the
Transaction constructor. Having derived classes pass necessary construction information up to base class constructors.

```c++
class Transaction {
public:
	explicit Transaction(const std::string& logInfo);
	void logTransaction(const std::string& logInfo) const; // now a non-virtual func
	...
};
Transaction::Transaction(const std::string& logInfo)
{
	...
	logTransaction(logInfo); // now a non-virtual call
}

class BuyTransaction: public Transaction {
public:
	BuyTransaction( parameters )
		: Transaction(createLogString( parameters )) // pass log info to base class constructor
	{ ... }
private:
	static std::string createLogString( parameters );
};
```

**The (private) static function createLogString**

* Using a helper function to create a value to pass to a base class constructor is often more convenient (and more readable).
* By making the function static, there's no danger of accidentally referring to the nascent BuyTransaction object's as-yet-uninitialized data
  members. 

***

## Item 10: Have assignment operators return a reference to *this
Assignment returns a reference to its left-hand argument, and that's the convention you should follow when you implement assignment operators for your classes

This convention applies to all assignment operators, not just the standard form.

```c++
class Widget {
public:
	...
	Widget& operator=(const Widget& rhs) // return type is a reference to the current class
	{ 
		...
		return *this; // return the lefthand object
	}
	Widget& operator+=(const Widget& rhs) // the convention applies to +=, -=, *=, etc.
	{
		...
		return *this;
	}
	Widget& operator=(int rhs) // it applies even if the operator's parameter type is unconventional
	{
		... 
		return *this;	
	}
	...
};
```

## Item 11: Handle assignment to self in operator=

In the implementation of operator=, you may fall into the trap of accidentally releasing a resource before you're done using it, if you don't consider the self assignment.

```c++
class Bitmap { ... };
class Widget {
	...
private:
	Bitmap *pb; // ptr to a heap-allocated object
};

Widget& Widget::operator=(const Widget& rhs) // unsafe impl. of operator=
{
	delete pb; // stop using current bitmap
	pb = new Bitmap(*rhs.pb); // start using a copy of rhs's bitmap
	return *this;
}
```

When *this (the target of the assignment) and rhs are the same object, the delete not only destroys the bitmap for the current object, it destroys the bitmap for rhs, too.



### The traditional way to prevent this error is to check for assignment to self via an identity test at the top of operator=

```c++
Widget& Widget::operator=(const Widget& rhs)
{
	if (this == &rhs) return *this; // identity test: if a self-assignment, do nothing
	delete pb;
	pb = new Bitmap(*rhs.pb);
	return *this;
}
```

This version continues to have exception trouble. In particular, if the “new Bitmap” expression yields an exception (either because there is insufficient memory for the allocation or because Bitmap's copy constructor throws one), the Widget will end up holding a pointer to a deleted Bitmap.

###  Making operator= exception-safe typically renders it self-assignment-safe, too. 
A careful ordering of statements can yield exception-safe (and self-assignment-safe) code. We just have to be careful not to delete pb until after we've copied what it points to.

```c++
Widget& Widget::operator=(const Widget& rhs)
{
	Bitmap *pOrig = pb; // remember original pb
	pb = new Bitmap(*rhs.pb);
	delete pOrig; // delete the original pb
	return *this;
}
```

### Use the technique known as “copy and swap.”

```c++
class Widget {
	...
	void swap(Widget& rhs); // exchange *this's and rhs's data;
};
Widget& Widget::operator=(const Widget& rhs)
{
	Widget temp(rhs); // make a copy of rhs's data
	swap(temp); // swap *this's data with the copy's
	return *this;
}
```

A variation on this theme：

```c++
Widget& Widget::operator=(Widget rhs) 	// rhs is a copy of the object passed in
{ 										// note pass by val
	swap(rhs); 							// swap *this's data with the copy's
	return *this;
}
```

It takes advantage of the facts that

* A class's copy assignment operator may be declared to take its argument by value
* Passing something by value makes a copy of it

## Item 12: Copy all parts of an object

In well-designed object-oriented systems that encapsulate the internal parts of objects, only two functions copy objects: the aptly named **copy constructor** and **copy assignment operator**. We'll call these the copying functions.

If you add a data member to a class, you need to make sure that you update the copying functions, too.

#### Copying functions in inheritance

```c++
void logCall(const std::string& funcName); // make a log entry
class Customer {
public:
	...
	Customer(const Customer& rhs);
	Customer& operator=(const Customer& rhs);
	...
private:
	std::string name;
};

class PriorityCustomer: public Customer { // a derived class
public:
	...
	PriorityCustomer(const PriorityCustomer& rhs);
	PriorityCustomer& operator=(const PriorityCustomer& rhs);
	...
private:
	int priority;
};
PriorityCustomer::PriorityCustomer(const PriorityCustomer& rhs)
		: priority(rhs.priority)
{
	logCall("PriorityCustomer copy constructor");
}
PriorityCustomer& PriorityCustomer::operator=(const PriorityCustomer& rhs)
{
	logCall("PriorityCustomer copy assignment operator");
	priority = rhs.priority;
	return *this;
}
```

The problem is that every PriorityCustomer also contains a copy of the data members it inherits from Customer, and those data members are not being copied at all.

Priority Customer's copy constructor specifies no arguments to be passed to its base class constructor (i.e., it makes no mention of Customer on its member initialization list), so **the Customer part of the PriorityCustomer object will be initialized by the Customer constructor taking no arguments** —by the default constructor. (Assuming it has one. If not, the code won't compile.) That constructor will perform a default initialization for name and lastTransaction.

Any time you take it upon yourself to write copying functions for a derived class, you must take care to also copy the base class parts. Those parts are
typically private, so **derived class copying functions must invoke their corresponding base class functions**

```c++
PriorityCustomer::PriorityCustomer(const PriorityCustomer& rhs)
	: Customer(rhs), // invoke base class copy ctor
	  priority(rhs.priority)
{
	logCall("PriorityCustomer copy constructor");
}
PriorityCustomer& PriorityCustomer::operator=(const PriorityCustomer& rhs)
{
	logCall("PriorityCustomer copy assignment operator");
	Customer::operator=(rhs); // assign base class parts
	priority = rhs.priority;
	return *this;
}
```

# 3. Resource Management

## Item 13: Use objects to manage resources

A resource is something that, once you're done using it, you need to return to the system. For example, dynamically allocated memory, file descriptors, mutex locks, fonts and brushes in graphical user interfaces (GUIs), database connections, and network sockets.

Suppose we have such class as below and a factory function: 

```c++
class Investment { ... }; // root class of hierarchy of investment types
Investment* createInvestment(); // return ptr to dynamically allocated
								// object in the Investment hierarchy;
								// the caller must delete it
void f()
{
	Investment *pInv = createInvestment(); // call factory function
	... // use pInv
	delete pInv; // release object
}
```

There are several ways f could fail to delete the investment object it gets from createInvestment:

* There might be a premature return statement somewhere inside the “...” part of the function
* If the uses of createInvestment and delete were in a loop, and the loop was prematurely exited by a break or goto statement
* Some statement inside the “...” might throw an exception

We can fix this problem by putting resources inside objects, and then we can rely on C++'s automatic destructor invocation to make sure that the resources are released.

The smart pointer's destructor automatically calls delete on what it points to:

```c++
void f()
{
	std::auto_ptr<Investment> pInv(createInvestment()); // call factory function
	... 												// use pInv as before
} // automatically delete pInv via auto_ptr's dtor

// auto_ptr在C++11中已经弃用，可用shared_ptr和unique_ptr代替
```

auto_ptrs have an unusual characteristic: copying them (via copy constructor or copy assignment operator) sets them to null, and the copying pointer assumes sole ownership of the resource!

An alternative to auto_ptr is a reference-counting smart pointer (RCSP). An RCSP is a smart pointer that keeps track of how many objects point to a
particular resource and automatically deletes the resource when nobody is pointing to it any longer.

*Both auto_ptr and tr1::shared_ptr use delete in their destructors, not delete []. That means that using auto_ptr or tr1::shared_ptr with dynamically allocated arrays is a bad idea*

### Two critical aspects of using objects to manage resources

* Resources are acquired and immediately turned over to resource managing objects.
* Resource-managing objects use their destructors to ensure that resources are released.

## Item 14: Think carefully about copying behavior in resource-managing classes
Sometimes, we need to create our own resource-managing classes when the resources are not heap-based.

```c++
void lock(Mutex *pm); // lock mutex pointed to by pm
void unlock(Mutex *pm); // unlock the mutex
```

To make sure that you never forget to unlock a Mutex you've locked, you'd like to create a class to manage locks.

```c++
class Lock {
public:
	explicit Lock(Mutex *pm): mutexPtr(pm) { lock(mutexPtr); } // acquire resource
	~Lock() { unlock(mutexPtr); } // release resource
private:
	Mutex *mutexPtr;
};

Mutex m; 			// define the mutex you need to use
...
{ 					// create block to define critical section
	Lock ml(&m); 	// lock the mutex
	... 			// perform critical section operations
} 					// automatically unlock mutex at end of block
```

The problem is that when a Lock object is copied, what should happen? Typically you have the choices:

* **Prohibit copying**

* **Reference-count the underlying resource**

  Copying an RAII*(Resource Acquisition Is Initialization)* object should increment the count of the number of objects referring to the resource.

  We can implement this behavior by containing a shared_ptr data member, and specify the deleter of shared_ptr

  ```c++
  class Lock {
  public:
  	explicit Lock(Mutex *pm) 		// init shared_ptr with the Mutex
  		: mutexPtr(pm, unlock) 		// to point to and the unlock func
  	{ 								// as the deleter
  		lock(mutexPtr.get());
  	}
  private:
  	std::tr1::shared_ptr<Mutex> mutexPtr; // use shared_ptr instead of raw pointer
  };
  ```

  The Lock class no longer declares a destructor. The synthetized destructor automatically invokes the destructor of the class's non-static data members. In this case, that is mutexPtr. And mutexPtr's destructor will automatically call its deleter -- unlock.

* **Copy the underlying resource**

  Copying the resource-managing object should also copy the resource it wraps. That is, copying a resource-managing object performs a “deep copy.”

* **Transfer ownership of the underlying resource**

  As auto_ptr, ownership of the resource is transferred from the copied object to the copying object.

## Item 15: Provide access to raw resources in resource managing classes
In a perfect world, you'd rely on resource managing classes for all your interactions with resources. But many APIs refer to resources directly, you'll have to bypass resource-managing objects and deal with raw resources. For example, tr1::shared_ptr and auto_ptr both offer a get member function to perform an explicit conversion, i.e., to return (a copy of) the raw pointer inside the smart pointer object.

Similarly because it is sometimes necessary to get at the raw resource inside an RAII object, some RAII class designers grease the skids by offering an conversion function.

```c++
FontHandle getFont(); 				// from C API—params omitted for simplicity
void releaseFont(FontHandle fh); 	// from the same C API

class Font { 	// RAII class
public:
	explicit Font(FontHandle fh) 	// acquire resource;
		: f(fh) 					// use pass-by-value
	{} 
	~Font() { releaseFont(f); } 	// release resource
private:
	FontHandle f; // the raw font resource
};
```

Assuming there's a large font-related C API that deals entirely with FontHandles, there will be a frequent need to convert from Font objects to FontHandles. 

* **The Font class could offer an explicit conversion function such as get:**

  ```c++
  class Font {
  public:
  	...
  	FontHandle get() const { return f; } // explicit conversion function
  	...
  };
  ```

* **The alternative is to have Font offer an implicit conversion function to its FontHandle**

  ```c++
  class Font {
  public:
  	...
  	operator FontHandle() const // implicit conversion function
  		{ return f; }
  ...
  };
  ```

Often, an explicit conversion function like get is the preferable path, because it minimizes the chances of unintended type conversions. Sometime, however, the naturalness of use arising from implicit type conversions will tip the scales in that direction

## Item 16: Use the same form in corresponding uses of new and delete
When you employ a new expression (i.e., dynamic creation of an object via a use of new), two things happen：

* memory is allocated
* one or more constructors are called for that memory

When you employ a delete expression (i.e., use delete), two other things happen:

* one or more destructors are called for the memory
* the memory is deallocated

If you use brackets in your use of delete(`delete []`), delete assumes an array is pointed to. Otherwise, it assumes that a single object is pointed to.

```c++
std::string *stringPtr1 = new std::string;
std::string *stringPtr2 = new std::string[100];
...
delete stringPtr1; // delete an object
delete [] stringPtr2; // delete an array of objects
```

if you used the “[]” form on stringPtr1, the result is undefined: delete would read some memory and interpret what it read as an array size, then start invoking that many destructors.

if you didn't use the “[]” form on stringPtr2, it would lead to too few destructors being called.

## Item 17: Store newed objects in smart pointers in standalone statements
Consider the code below:

```c++
int priority();
void processWidget(std::tr1::shared_ptr<Widget> pw, int priority);
processWidget(new Widget, priority());		// error: shared_ptr's constructor taking a raw pointer is explicit 
processWidget(std::tr1::shared_ptr<Widget>(new Widget), priority());	// ok
```

The last statement will compile, and before processWidget can be called, compilers must generate code to do these three things:

* Call priority
* Execute “new Widget”
* Call the tr1::shared_ptr constructor

The “`new Widget`” expression must be executed before the tr1::shared_ptr constructor can be called, but **the call to priority can be performed first, second, or third in C++.**

If If compilers choose to perform it second, we end up with this sequence of operations:

* Execute “new Widget”
* Call priority
* Call the tr1::shared_ptr constructor

Here that is the problem, if the call to priority yields an exception, the pointer returned from “new Widget” will be lost, because it won't have been stored in the tr1::shared_ptr we were expecting.

**A leak in the call to processWidget can arise because an exception can intervene between the time <font color='red'>a resource is created</font> (via “new Widget”) and the time that <font color='red'>resource is turned over to a resource-managing object</font>.**

The way to avoid problems like this is simple: use a separate statement to create the Widget and store it in a smart pointer, then pass the smart pointer to processWidget:

```c++
std::tr1::shared_ptr<Widget> pw(new Widget); // store newed object in a smart pointer in a standalone statement
processWidget(pw, priority()); // this call won't leak
```

# 4. Designs and Declarations

## Item 18: Make interfaces easy to use correctly and hard to use incorrectly

Developing interfaces that are easy to use correctly and hard to use incorrectly requires that you consider the kinds of mistakes that clients might make.

For example, suppose you're designing the constructor for a class representing dates in time.

```c++
class Date {
public:
	Date(int month, int day, int year);
	...
};

Date d(30, 3, 1995); // Oops! Should be "3, 30" , not "30, 3"
Date d(3, 40, 1995); // Oops! Should be "3, 30" , not "3, 40"
```

Many client errors can be prevented by the introduction of **new types**. The judicious introduction of new types can work wonders for the prevention of interface usage errors: 

```c++
struct Day {
    explicit Day(int d)
        :val(d) {}
    int val;
};
struct Month {
    explicit Month(int m)
        :val(m) {}
    int val;
};
struct Year{
	explicit Year(int y)
    	:val(y) {}
	int val;
};
class Date {
public:
	Date(const Month& m, const Day& d, const Year& y);
	...
};

Date d(30, 3, 1995); // error! wrong types
Date d(Day(30), Month(3), Year(1995)); // error! wrong types
Date d(Month(3), Day(30), Year(1995)); // okay, types are correct
```

### Predefine the set of all valid Months

Moreover, there are only 12 valid month values, so the Month type should reflect that. One way to do this would be to use an enum to represent the month, but enums are not as type-safe as we might like. For example, enums can be used like ints. A safer solution is to predefine the set of all valid Months:

```c++
class Month {
public:
	static Month Jan() { return Month(1); } 	// functions returning all valid Month values
	static Month Feb() { return Month(2); } 	// using functions instead of objects to represent specific months
												// because reliable initialization of non-local static objects can be 			
    											// problematic
	... 										
	static Month Dec() { return Month(12); } 	
	... // other member functions
private:
	explicit Month(int m); // prevent creation of new Month values
	...
};
Date d(Month::Mar(), Day(30), Year(1995));
```

### Restrict what can be done with a type

A common way to impose restrictions is to add const. **Unless there's a good reason not to, have your types behave consistently with the built-in types**.

For example, const-qualifying the return type from operator* can prevent clients from making this error for user-defined types:

```c++
if (a * b = c) ... // oops, meant to do a comparison!
```

### Any interface that requires that clients remember to do something is prone to incorrect use
```c++
Investment* createInvestment(); // from Item 13; parameters omitted for simplicity
```

Item 13 shows how clients can store createInvestment's return value in a smart pointer like auto_ptr or tr1::shared_ptr, thus turning over to the smart pointer the responsibility for using delete. But what if clients forget to use the smart pointer? We should have the factory function return a smart pointer in the first place instead:

```c++
tr1::shared_ptr<Investment> createInvestment(); // from Item 13; parameters omitted for simplicity
```

Moreover, suppose clients who get an Investment* pointer from createInvestment are expected to pass that pointer to a function called getRidOfInvestment instead of using delete on it. The implementer of createInvestment can forestall such problems by returning a tr1::shared_ptr with
getRidOfInvestment bound to it as its deleter.

```c++
std::tr1::shared_ptr<Investment> createInvestment()
{
	std::tr1::shared_ptr<Investment> retVal(static_cast<Investment*>(0), getRidOfInvestment);
	retVal = ... ; // make retVal point to the correct object
	return retVal;
}
```

An especially nice feature of tr1::shared_ptr is that it automatically uses its per-pointer deleter to eliminate another potential client error, the “cross-DLL
problem.” This problem crops up when an object is created using new in one dynamically linked library (DLL) but is deleted in a different DLL. tr1::shared_ptr avoids the problem, because its default deleter uses delete from the same DLL where the tr1::shared_ptr is created.

## Item 19: Treat class design as type design

In C++, defining a new class defines a new type. You should therefore approach class design with the same care that language designers lavish on the design of the language's built-in types.

When you design your class, consider the following questions:

* **How should objects of your new type be created and destroyed?**

  The design of your class's constructors and destructor, as well as its memory allocation and deallocation functions (operator new, operator new[], operator delete, and operator delete[])

* **How should object initialization differ from object assignment?**

  The differences between your constructors and your assignment operators

* **What does it mean for objects of your new type to be passed by value?**

  <font color='red'>The copy constructor defines how pass-by-value is implemented for a type.</font>

* **What are the restrictions on legal values for your new type?**

  The error checking you'll have to do inside your member functions

  The exceptions your functions throw

* **Does your new type fit into an inheritance graph?**

  If you inherit from existing classes, you are constrained by the design of those classes

  If you wish to allow other classes to inherit from your class, that affects whether the functions you declare are virtual, especially your destructor

* **What kind of type conversions are allowed for your new type?**

  * If you wish to allow objects of type T1 to be implicitly converted into objects of type T2
    * Write a type conversion function in class T1 (e.g., operator T2)
    * Write nonexplicit constructor in class T2 that can be called with a single argument
  * If you wish to allow explicit conversions only
    * Write functions to perform the conversions

* **What operators and functions make sense for the new type?**

  Which functions you'll declare for your class

* **What standard functions should be disallowed?**

  Those are the ones you'll need to declare private

* **Who should have access to the members of your new type?**

  This question helps you determine which members are public, which are protected, and which are private.

  It also helps you determine which classes and/or functions should be friends, as well as whether it makes sense to nest one class inside another.

* **How general is your new type?**

  Perhaps you're defining a whole family of types. If so, you don't want to define a new class, you want to define a new **class template**

* **Is a new type really what you need?**

  If you're defining a new derived class only so you can add functionality to an existing class, perhaps you'd better achieve your goals by simply defining one or more non-member functions or templates.

## Item 20: Prefer pass-by-reference-to-const to pass-by value

Unless you specify otherwise, function parameters are initialized with copies of the actual arguments, and function callers get back a copy of the value returned by the function. **These copies are produced by the objects' copy constructors.** Moreover, after the function ends, each constructor call is matched by a destructor call.

Pass by reference-to-const is much more efficient: no constructors or destructors are called, because
no new objects are being created.

### Passing parameters by reference also avoids the slicing problem.

When a derived class object is passed (by value) as a base class object, the base class copy constructor is called, and the specialized features that make the object behave like a derived class object are “sliced” off.

```c++
class Window {
public:
	...
	std::string name() const; // return name of window
	virtual void display() const; // draw window and contents
};
class Window-WithScrollBars: public Window {
public:
	...
	virtual void display() const;
};
```

Now suppose you'd like to write a function to print out a window's name and then display the window. Here's the wrong way to write such a function

```c++
void printNameAndDisplay(Window w) // incorrect! parameter may be sliced!
{
	std::cout << w.name();
	w.display();
}
Window-WithScrollBars wwsb;
printNameAndDisplay(wwsb);
```

The parameter w will be constructed as a Window object, Inside printNameAndDisplay, w will always act like an object of class Window, regardless of the type of object passed to the function. In particular, the call to display inside printNameAndDisplay will always call Window::display, never Window-WithScrollBars::display.

```c++
void printNameAndDisplay(const Window& w) // fine, parameter won't be sliced
{
	std::cout << w.name();
	w.display();
}
```

### For built-in types and STL iterator and function object types, pass-by-value is usually appropriate

References are typically implemented as pointers, so passing something by reference usually means really passing a pointer.

If you have an object of a built-in type (e.g., an int), it's often more efficient to pass it by value than by
reference.

This same advice applies to iterators and function objects in the STL, because, by convention, they are designed to be passed by value.

### It doesn't mean that all small types are good candidates for pass-by-value, even if they're user-defined.

* Just because an object is small doesn't mean that calling its copy constructor is inexpensive. Many objects — most STL containers among them — contain little more than a pointer, but copying such objects entails copying everything they point to. That can be very expensive.
* Some compilers treat built-in and user-defined types differently. They refuse to put objects consisting of only a double into a register, but compilers will certainly put pointers (the implementation of references) into registers.
* Being user-defined, their size is subject to change. A type that's small now may be bigger in a future release, because its internal implementation may change.

## Item 21: Don't try to return a reference when you must return an object
A function can create a new object in only two ways: on the stack or on the heap.

### Creation on the stack is accomplished by defining a local variable.

```c++
class Rational {
public:
	Rational(int numerator = 0, int denominator = 1);
	...
private:
	int n, d; // numerator and denominator
friend const Rational operator*(const Rational& lhs, const Rational& rhs);
};

const Rational& operator*(const Rational& lhs, const Rational& rhs) // warning! bad code!
{
	Rational result(lhs.n * rhs.n, lhs.d * rhs.d);
	return result;
}
```

This function returns a reference to result, but result is a local object, and local objects are destroyed when the function exits. As a result, it returns a reference to an ex-Rational; a former Rational; the empty, stinking, rotting carcass of what used to be a Rational but is no longer, because it has been destroyed.

### Constructing an object on the heap and returning a reference to it
```c++
const Rational& operator*(const Rational& lhs, const Rational& rhs) // warning! more bad code!
{
	Rational *result = new Rational(lhs.n * rhs.n, lhs.d *rhs.d);
	return *result;
}
```

You will have a different problem: who will apply delete to the object conjured up by your use of new?

### An implementation based on operator* returning a reference to a static Rational object, one defined inside the function

```c++
const Rational& operator*(const Rational& lhs, const Rational& rhs) // warning! yet more bad code!
{
	static Rational result; // static object to which a reference will be returned
	result = ... ; // multiply lhs by rhs and put the product inside result
	return result;
}
```

Consider this perfectly reasonable client code

```c++
bool operator==(const Rational& lhs, const Rational& rhs);// an operator== for Rationals
Rational a, b, c, d;
...
if ((a * b) == (c * d)){
	// do whatever's appropriate when the products are equal;
}
else{
	// do whatever's appropriate when they're not;
}
```

The expression ((a*b) == (c*d)) will always evaluate to true, regardless of the values of a, b, c, and d!

The equivalent functional form of `(a * b) == (c * d)` is `operator==( operator*(a, b) , operator*(c, d) )` 

Notice that when operator== is called, there will already be two active calls to operator*, each of which will return a reference to the static Rational object
inside operator\*. Thus, operator== will be asked to compare the value of the static Rational object inside operator\*

### The right way to write a function that must return a new object is to have that function return a new object.
```c++
inline const Rational operator*(const Rational& lhs, const Rational& rhs)
{
	return Rational(lhs.n * rhs.n, lhs.d * rhs.d);
}
```

***

## Item 22: Declare data members `private`

If data members aren't public, the only way for clients to access an object is via member functions. And if you use functions to get or set data members' value, you can implement no access, read-only access, and read-write access. Heck, you can even implement write-only access if you want to:

```c++
class AccessLevels {
public:
	...
	int getReadOnly() const { return readOnly; }
	void setReadWrite(int value) { readWrite = value; }
	int getReadWrite() const { return readWrite; }
	void setWriteOnly(int value) { writeOnly = value; }
private:
	int noAccess; // no access to this int
	int readOnly; // read-only access to this int
	int readWrite; // read-write access to this int
	int writeOnly; // write-only access to this int
};
```

### Encapsulation

If you hide your data members from your clients (i.e., encapsulate them), you can ensure that class invariants are always maintained, because
only member functions can affect them. Furthermore, you reserve the right to change your implementation decisions later.

### `protected` data members

Suppose we have a public data member, and we eliminate it. How much code might be broken? All the client code that uses it, which is generally *an*
*unknowably large amount*.

Suppose we have a protected data member, and we eliminate it. How much code might be broken now? All the derived classes that use it, which is, again, typically *an unknowably large amount* of code.

**Protected data members are thus as unencapsulated as public ones**, because in both cases, if the data members are changed, an unknowably large amount of client code is broken.

***

## Item 23: Prefer non-member non-friend functions to member functions
Imagine a class for representing web browsers with a set of member functions:

```c++
class WebBrowser {
public:
	...
	void clearCache();
	void clearHistory();
	void removeCookies();
	...
};
```

Many users will want to perform all these actions together, so WebBrowser might also offer a function to do just that:

```c++
class WebBrowser {
public:
	...
	void clearEverything(); // calls clearCache, clearHistory, and removeCookies
	...
};
```

Of course, this functionality could also be provided by a non-member function that calls the appropriate member functions：

```c++
void clearBrowser(WebBrowser& wb)
{
	wb.clearCache();
	wb.clearHistory();
	wb.removeCookies();
}
```

### Encapsulation

Consider the data associated with an object. We can count the number of functions that can access that data: **the more functions that can access it, the less encapsulated the data**.

For data members that are private, the number of functions that can access them is the number of **member functions of the class** plus the number of **friend functions**, because only members and friends have access to private members.

Given a choice between a member function and a non-member non-friend function providing the same functionality, the choice **yielding greater encapsulation** is the **non-member non-friend function**, because it doesn't increase the number of functions that can access the private parts of the class.

### Make `clearBrowser` a nonmember function in the same namespace as WebBrowser

```c++
namespace WebBrowserStuff {
	class WebBrowser { ... };
	void clearBrowser(WebBrowser& wb);
	...
}
```

namespaces, unlike classes, can be spread across multiple source files.

A class like `WebBrowser` might have a large number of convenience functions, some related to bookmarks, others related to printing, still others related to cookie management, etc.

The straightforward way to separate them is to declare bookmark-related convenience functions in one header file, cookie-related convenience functions in a different header file, printing-related convenience functions in a third, etc.:

```c++
// header "webbrowser.h" — header for class WebBrowser itself
// as well as "core" WebBrowser-related functionality
namespace WebBrowserStuff {
	class WebBrowser { ... };
	... // "core" related functionality, e.g.
		// non-member functions almost
		// all clients need
} 

// header "webbrowserbookmarks.h"
namespace WebBrowserStuff {
	... // bookmark-related convenience functions
}

// header "webbrowsercookies.h"
namespace WebBrowserStuff {
	... // cookie-related convenience functions
}
...
```

Partitioning functionality in this way is not possible when it comes from a class's member functions, because a class must be defined in its entirety.

**Putting all convenience functions in multiple header files — but one namespace** — also means that clients can easily extend the set of convenience
functions. All they have to do is add more non-member non-friend functions to the namespace. This is another feature classes can't offer, because class definitions are closed to extension by clients.

***

## Item 24: Declare non-member functions when type conversions should apply to all parameters

Having classes support implicit type conversions is generally a bad idea, Of course, there are exceptions to this rule, when creating **numerical types**

```c++
class Rational {
public:
	Rational(int numerator = 0, nt denominator = 1);  	// ctor is deliberately not explicit; 																		// allows implicit int-to-Rational conversions	
	int numerator() const; 								// accessors for numerator and denominator
	int denominator() const;
private:
	...
};
```

### Making operator* a member function of Rational

```c++
class Rational {
public:
	...
	const Rational operator*(const Rational& rhs) const;
};

Rational oneEighth(1, 8);
Rational oneHalf(1, 2);
Rational result = oneHalf * oneEighth; 	// fine
result = result * oneEighth; 			// fine
```

**When you try to do mixed-mode arithmetic, however, you find that it works only half the time:**

```c++
result = oneHalf * 2; // fine
result = 2 * oneHalf; // error!

//The equivalent fuction form:
result = oneHalf.operator*(2); // fine
result = 2.operator*(oneHalf); // error!
```

In the form `oneHalf.operator*(2)`, Compilers know you're passing an int and that the function requires a Rational, but they also know they can conjure up a suitable Rational by calling the Rational constructor with the int you provided, so that's what they do. Of course, compilers do this only because a non-explicit constructor is involved.

Parameters are eligible for implicit type conversion only if they are listed in the parameter list.

### Making operator* a non-member function

```c++
class Rational {
	... // contains no operator*
};
const Rational operator*(const Rational& lhs, const Rational& rhs)// now a non-member function
{
	return Rational(lhs.numerator() * rhs.numerator(), lhs.denominator() * rhs.denominator());
}
Rational oneFourth(1, 4);
Rational result;
result = oneFourth * 2; // fine
result = 2 * oneFourth; // hooray, it works!
```

If you need type conversions on all parameters to a function (including the one that would otherwise be pointed to by the
this pointer), the function must be a non-member.

## Item 25: Consider support for a non-throwing `swap`

By default, swapping is accomplished via the standard swap algorithm. Its typical implementation is exactly what you'd expect:

```c++
namespace std {
	template<typename T> // typical implementation of std::swap;
	void swap(T& a, T& b) // swaps a's and b's values
	{
		T temp(a);
		a = b;
		b = temp;
	}
}
```

As long as your types support copying (**via copy constructor and copy assignment operator**), the default `swap` implementation will let objects of your types be swapped.

However, the default swap implementation involves copying three objects: a to temp, b to a, and temp to b. For some types, none of these copies are really necessary:

```c++
class WidgetImpl { 	// class for Widget data;
public: 			// details are unimportant
	...
private:
	int a, b, c; 	// possibly lots of data 
	std::vector<double> v; // expensive to copy!
	...
};
```

```c++
class Widget { 						// class using the pimpl idiom
public:
	Widget(const Widget& rhs);
	Widget& operator=(const Widget& rhs) 	// to copy a Widget, copy its WidgetImpl object
	{
		... 
		*pImpl = *(rhs.pImpl); 
	}
	...
private:
	WidgetImpl *pImpl; // ptr to object with this
};
```

> **pimpl idiom**: pointer to implementation
>
> Make a class consist primarily of a pointer to another type that contains the real data

To swap the value of two Widget objects, all we really need to do is swap their pImpl pointers, but the default swap algorithm would copy not only three Widgets, but also three WidgetImpl objects.

### Specialize std::swap for Widget

```c++
namespace std {
	template<> // this is a specialized version of std::swap when T is Widget
	void swap<Widget>(Widget& a, Widget& b)
	{
		swap(a.pImpl, b.pImpl); // to swap Widgets, swap their pImpl pointers;
	}
	// this won't compile
}
```

In general, we're not permitted to alter the contents of the std namespace, but we are allowed to **totally** specialize standard templates (like swap) for types of our own creation (such as Widget).

**However this function won't compile.** That's because it's trying to access the pImpl pointers inside a and b, and they're private.

### Have Widget declare a public member function called swap that does the actual swapping, then specialize std::swap to call the member function

```c++
class Widget { 	// same as above, except for the addition of the swap mem func
public:
	...
	void swap(Widget& other)
	{
		using std::swap; // the need for this declaration is explained later in this Item
		swap(pImpl, other.pImpl); // to swap Widgets, swap their pImpl pointers
	} 
	...
};

namespace std {
	template<> // revised specialization of std::swap
	void swap<Widget>(Widget& a, Widget& b)// 
	{
		a.swap(b); // to swap Widgets, call their swap member function
	}
}
```

Not only does this compile, it's also consistent with the STL containers. All of STL containers provide both public swap member functions and versions of std::swap that call these member functions.

### What if the `WidgetImpl` and `Widget` class are template class

```c++
template<typename T>
class WidgetImpl { ... };
template<typename T>
class Widget { ... };
```

Putting a swap member function in Widget (and, if we need to, in WidgetImpl) is as easy as before, but we run into trouble with the specialization for std::swap:

```c++
namespace std {
	template<typename T>
	void swap<Widget<T> >(Widget<T>& a, Widget<T>& b)// error! illegal code!
	{
        a.swap(b);
    }
}
```

We're trying to **partially** specialize a function template (std::swap), but C++ allows partial specialization of class templates, it doesn't allow it for function templates.

#### When you want to “partially specialize” a function template, the usual approach is to **add an overload**：

```c++
namespace std {
	template<typename T> // an overloading of std::swap
	void swap(Widget<T>& a, Widget<T>& b)// (note the lack of "<...>" after "swap"), but see below for why this isn't valid code
	{
        a.swap(b);
    }
}
```

It's okay to totally specialize templates in std, but it's not okay to add new templates (or classes or functions or anything else) to std.

What we should do is to declare a non-member swap that calls the member swap, we just don't declare the non-member to be a specialization or overloading of std::swap:

```c++
namespace WidgetStuff {
	... // templatized WidgetImpl, etc.
	template<typename T> 					// as before, including the swap
	class Widget { ... }; 					// member function
	...
	template<typename T> 					// non-member swap function;
	void swap(Widget<T>& a, Widget<T>& b) 	// not part of the std namespace
	{
		a.swap(b);
	}
}
```

#### From a client's point of view

Suppose you're writing a function template where you need to swap the values of two objects, Which swap should this call? 

* The general one in std, which you know exists;
* A specialization of the general one in std, which may or may not exist;
* A T-specific one, which may or may not exist and which may or may not be in a namespace (but should certainly not be in std)

What you desire is to call a T-specific version if there is one, but to fall back on the general version in std if there's not:

```c++
template<typename T>
void doSomething(T& obj1, T& obj2)
{
	using std::swap; // make std::swap available in this function
	...
	swap(obj1, obj2); // call the best swap for objects of type T
	...
}
```

C++'s name lookup rules ensure that this will find any T-specific swap at global scope or in the same namespace as the type T:

* If T is Widget in the namespace WidgetStuff, compilers will use argument-dependent lookup to find swap in WidgetStuff.
* If no T-specific swap exists, compilers will use swap in std, thanks to the using declaration that makes std::swap visible in this function.
* Even then, compilers will prefer a T-specific specialization of std::swap over the general template, so if std::swap has been specialized for T, the specialized version will be used.

### Summary

If the default implementation of swap isn't efficient enough, do the following:

* Offer a public swap member function that efficiently swaps the value of two objects of your type. For reasons I'll explain in a moment, this
  function should never throw an exception.
* Offer a non-member swap in the same namespace as your class or template. Have it call your swap member function.
* If you're writing a class (not a class template), specialize std::swap for your class. Have it also call your swap member function.

Finally, if you're calling swap, be sure to include a using declaration to make std::swap visible in your function, then call swap without any namespace
qualification.

# 5. Implementations

## Item 26: Postpone variable definitions as long as possible

Consider the following function, which returns an encrypted version of a password:

```c++
// this function defines the variable "encrypted" too soon
std::string encryptPassword(const std::string& password)
{
	using namespace std;
	string encrypted;
	if (password.length() < MinimumPasswordLength) {
		throw logic_error("Password is too short");
	}
	... // do whatever is necessary to place an
		// encrypted version of password in encrypted
	return encrypted;
}
```

The object `encrypted` is unused if an exception is thrown. That is, you'll pay for the construction and destruction of encrypted even if encryptPassword throws an exception.

```c++
// this function postpones encrypted's definition until
// it's necessary, but it's still needlessly inefficient
std::string encryptPassword(const std::string& password)
{
	using namespace std;
	if (password.length() < MinimumPasswordLength) {
		throw logic_error("Password is too short");
	}
	string encrypted;	// default-construct encrypted
    					// encrypted is defined without any initialization arguments. 
						// That means its default constructor will be used.
	encrypted = password; // assign to encrypted
	encrypt(encrypted);
	return encrypted;
}
```

```c++
// finally, the best way to define and initialize encrypted: 
// skipping the pointless and potentially expensive default construction
std::string encryptPassword(const std::string& password)
{
	... // import std and check length
	string encrypted(password); // define and initialize via copy constructor
	encrypt(encrypted);
	return encrypted;
}
```

Remember that default-constructing an object and then assigning to it is less efficient than initializing it with the value you really want it to have.

**As a result, not only should you postpone a variable's definition until right before you have to use the variable, you should also try to postpone the definition until you have initialization arguments for it.**

### variable used in a loop

```c++
// Approach A: define outside loop
Widget w;
for (int i = 0; i < n; ++i){ 
	w = some value dependent on i; 
	... 
}
// Approach B: define inside loop
for (int i = 0; i < n; ++i){
    Widget w(some value dependent on i);
    ...;
}
```

* Approach A: 1 constructor + 1 destructor + n assignments.
* Approach B: n constructors + n destructors.

For classes where an assignment costs less than a constructor-destructor pair, Approach A is generally more efficient.

However, Approach A makes the name w visible in a larger scope, something that's contrary to program comprehensibility and maintainability.

As a result, unless you know that (1) assignment is less expensive than a constructor-destructor pair and (2) you're dealing with a performance-sensitive part of your code, you should default to using Approach B.

## Item 27: Minimize casting.

### Casting syntax

#### old-style casts 

* **C-style casts:**

  ```c++
  (T) expression // cast expression to be of type T
  ```

* **Function-style casts**

  ```c++
  T(expression) // cast expression to be of type T
  ```

There is no difference in meaning between these forms

#### new-style casts

* **`const_cast<T>(expression)`**

  `const_cast` is typically used to cast away the constness of objects. It is the only C++style cast that can do this.

* **`dynamic_cast<T>(expression)`**

  `dynamic_cast` is primarily used to perform “safe downcasting,” i.e., to determine whether an object is of a particular type in an inheritance
  hierarchy. It is the only cast that cannot be performed using the old-style syntax.

  <font color='red'>It is also the only cast that may have a significant runtime cost.</font>

* **`reinterpret_cast<T>(expression)`**

  `reinterpret_cast` is intended for low-level casts that yield implementation dependent (i.e., unportable) results, e.g., casting a pointer to an int.

* **`static_cast<T>(expression)`**

  `static_cast` can be used to force implicit conversions (e.g., non-const object to const object, int to double, etc.). 

  It can also be used to perform the reverse of many such conversions (e.g., void* pointers to typed pointers, pointer-to-base to pointer-to-derived), though it cannot cast from const to non-const objects. (Only const_cast can do that.)

**The old-style casts continue to be legal, but the new forms are preferable:**

* They're much easier to identify in code, thus simplifying the process of finding places in the code where the type system is being subverted.
* The more narrowly specified purpose of each cast makes it possible for compilers to diagnose usage errors. For example, if you try to cast away constness using a new-style cast other than const_cast, your code won't compile.

About the only time I use an old-style cast is when I want to call an explicit constructor to pass an object to a function:

```c++
class Widget {
public:
	explicit Widget(int size);
	...
};
void doSomeWork(const Widget& w);

doSomeWork(Widget(15)); // create Widget from int with functionstyle cast
doSomeWork(static_cast<Widget>(15)); // create Widget from int with C++style cast
```

They do exactly the same thing here: create a temporary Widget object to pass to doSomeWork.

### Casting in an inheritance hierarchy
```c++
class Window { // base class
public:
	virtual void onResize() { ... } // base onResize impl
	...
};
class SpecialWindow: public Window { // derived class
public:
	virtual void onResize() { 		// derived onResize impl;
		static_cast<Window>(*this).onResize(); // cast *this to Window, then call its onResize;
		// this doesn't work!
		... // do SpecialWindow-specific stuff
	} 
	...
};
```

The code casts *this to a Window. The resulting call to onResize therefore invokes Window::onResize.

<font color='red'>However it does not invoke that function on the current object! Instead, the cast creates a new, temporary copy of the base class part of *this, then invokes onResize on the copy!</font> If Window::onResize modifies the current object, the current object won't be modified. Instead, a copy of that object will be modified.

The solution is to eliminate the cast, replacing it with what you really want to say:

```c++
class SpecialWindow: public Window {
public:
	virtual void onResize() {
		Window::onResize(); // call Window::onResize on *this
        ...
	}
	...
};
```

### Many implementations of dynamic_cast can be quite slow

The need for dynamic_cast generally arises because you want to perform derived class operations on what you believe to be a derived class object, but
you have only a pointer- or reference-to-base.

For example, if, in our Window/SpecialWindow hierarchy, only SpecialWindows support blinking

```c++
class Window { ... };
class SpecialWindow: public Window {
public:
	void blink();
	...
};
typedef  std::vector<std::tr1::shared_ptr<Window> > VPW;
VPW winPtrs;
...
for (VPW::iterator iter = winPtrs.begin(); iter != winPtrs.end(); ++iter) {
	if (SpecialWindow *psw = dynamic_cast<SpecialWindow*>(iter->get()))
		psw->blink();
}
```

Avoid casts whenever practical, especially dynamic_casts in performance-sensitive code. If a design requires casting, try to develop a cast-free alternative.

There are two general ways to avoid this problem:

#### Use containers that store pointers (often smart pointers) to derived class objects directly
```c++
typedef std::vector<std::tr1::shared_ptr<SpecialWindow> > VPSW;
VPSW winPtrs;
...
for (VPSW::iterator iter = winPtrs.begin(); iter != winPtrs.end(); ++iter)
	(*iter)->blink();
```

Of course, this approach won't allow you to store pointers to all possible Window derivatives in the same container.

#### Provide virtual functions in the base class

Though only SpecialWindows can blink, maybe it makes sense to declare the function in the base class, offering a default implementation that does nothing:

```c++
class Window {
public:
	virtual void blink() {} // default impl is no-op;
	...
}; 
class SpecialWindow: public Window {
public:
	virtual void blink() { ... }; // in this class, blink does something
    ...
};

typedef std::vector<std::tr1::shared_ptr<Window> > VPW;
VPW winPtrs; // container holds (ptrs to) all possible Window types
for (VPW::iterator iter = winPtrs.begin(); iter != winPtrs.end(); ++iter)
	(*iter)->blink();
```

