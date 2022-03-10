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

