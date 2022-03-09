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

### Avoiding Duplication in const and Non-const Member Functions