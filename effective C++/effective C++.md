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

