# Week 1 Wednesday Pre-Lecture

## Why we don't write public data members

1. People could change your data when they shouldn't
2. It makes it difficult to change the implementation without changing the interface

## `const` Member Functions

When you're designing the class, you should consider which functions will and won't modify the underlying function. Mark those functions `const` to promise they won't change anything.

## Alternate Constructor Stucture

Remember from CS31:

```cpp
Circle::Circle(double x, double y, double r) 
: m_x(x), m_y(y), m_r(r) {
    // you can still do stuff in the constructor body here
    if (r < 0) {
        cout << "bad radius";
        exit(-1);
    }
}
```

## Order of Construction

1. haven't learned this yet
2. Construct the data members
   - This consults the member initialization list
   - If a data member is not listed in the member initialization list:
     - If it's a builtin type, it's left uninitialized
     - If it's a class type, the class's default constructor is called (so it won't compile if this constructor doesn't exist)
   - Sometimes, the order in which the data members are initialized matters. The canonical order should be the order in which they are declared in the class declaration. This should (ideally) match with the order of the members in the member initialization list.
3. Execute the body of the constructor

## Separate Compilation

When we're working with large programs, we sometimes want to make small changes without recompiling the whole thing. Thus, we compile components separately and then link them together.

This leads to our system of our headers: we put all the declarations in one place. We use:
```cpp
#include "Circle.h"
```
to "put" all the code into our `.cpp` file and allow the different functions/classes to be used.

There are two main steps of compilation:
1. Translating all your .cpp files into machine code. (`.cpp` -> `.o`)
   1. This includes lists of names it defines and names it needs
2. Then all the lists are matched up and files are **linked** together so that every name is defined. It also goes into the library to find what it needs.
   1. Must have just one `main()` routine
   2. Nothing can be defined twice (don't put definitions in the header files, these will get `#include`-d in multiple places)

### Header File Include Practices

Just a general principle: if your code in a particular file needs something, just include it (they're protected from multiple inclusion), even if it includes something that already has it. This is just a good habit in the case that other files change.

Don't do `using namespace std;` in your header files (conflicting names and forcing to specify namespace names).

Another small note is that `<cstdlib>` includes the `exit()` function that Smallberg uses.

```cpp
#include "myfile.h" // specify user-written declaration file
                    // the name will be treated as a path
#include <iostream> 
// standard c++ libraries use this convention
```

# Week 1 Wednesday Lecture

`const` to the left generally does nothing. Unless it denotes the return of a pointer to a value you can't modify like `const Foo * h;` in a function declaration (called like `obj.h()`). 
You can't do `obj.h()->modify();` (not marked const function) because it modifies the object at end of ptr.

## Constants, pointers, and constant pointers

```cpp
const Foo * h(const double * p) const;
```
1. return pointer to const
2. parameter is const ptr but you can modify the thing at the end of the arrow
3. "this" ptr is a constant obj


Pointers to `int` can't be pointed at `const int`. Because the pointer records whether or not it can change the thing it points at. However, if you declare a `const int *`, you can point it at either an `int` or a `const int`. If it points at a normal `int`, you still can't modify.
`const int * pi` can still be pointed at different things, so the pointer itself is not "const" perse. `const int * const cpi = &ci;` is something that you can't repoint, and it points to a const object.

What does `const` mean to left and right of star?
- left of star refers to thing it points to
- right of star refers to the name of the thing we're declaring

`int& ri = i;` binds reference to some kind of int (generally as param, not declaration)
- makes ri another name for i
- can't make ri refer to any other int later
- so basically like a constant pointer
- except you can ri = 10; just like with i

## Default Constructors

With no default constructor, you can't make a built-in array of those objects. You can make an array of pointers to objects without default constructors. It can be dumb to have default. constructor called 800 times to do `Blah a[800];`

```cpp
// is this better?
Blah * bpa[1000];
int nBlahs = 0;
// then later...
pba[nBlahs] = new Blah(x, y);
nBlahs++;
```
Conclusion: don't have a default constructor if it doesn't make sense.

## `new` and Variable Lifecycle

- usage of `new` is based on lifetime of object
- local variables, "automatic", "on the stack", has lifetime of function
- global variables, (outside of any function), has lifetime of program
- dynamically allocated variables, "on the heap", created using new, return a pointer to place on heap
  - only go away when you explicitly call delete
  - delete takes a pointer to the object you want to delete
  - the value of the pointer doesn't change, "dangling pointer", undefined behavior to follow pointer
  - undefined behavior to call delete again
- if you return a pointer that pointed to a local object
- the local object gets destroyed so a dangling pointer is returned
- you don't use `delete` on pointers to local variables