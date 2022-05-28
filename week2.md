# Week 2 Monday Lecture

## Separate Compilation Issues

For each source file (.cpp) in a project, compiler produces object file (.o or .obj) containing:
- the machine language translation of the code
- storage for global objects (like `std::cout`)
- a list of global names defined (aka implemented) by this object file
- a list of global names used in this file that need a definition somewhere

Then, the linker brings all the objects together with object files from libraries to make an executable file. The linker needs these rules to be upheld:
- Nothing can be defined more than once
- Every need must be satisfied by some definition
- There must be exactly one main routine

Simple rule for usage:
> If you want to use a class type, include the appropriate header

## Example Project

Point.h
```cpp
// some include guards:
// (still the only portable way to protect)
#ifdef POINT_INCLUDED
#define POINT_INCLUDED
// so it will only define once, as desired

class Point {
  //...
};

#endif // POINT_INCLUDED
```
Circle.h
```cpp
#include "Point.h"

class Circle {
  //...
  private:
    Point m_center;
    double m_radius;
};
```
main.cpp
```cpp
#include "Circle.h"
// shouldn't have to include "Point.h" if not being used directly, but also should require it if it is being used.
// you shouldn't have to know if Circle is implemented with Point or not

//if I #include "Point.h" at this point, now we'll get a complication error because Point will get declared twice (not with the include guards I included)

int main() {
  Circle c(-2, 5, 10);
  return 0;
}
```

## Include Guard Rules

 - Make the symbol you're defining to be unique
   -  Unique names typically include filename
 - Don't write anything significant outside the `#ifndef` block

## Pointer Example

This works with the [Incomplete Type Declaration](#incomplete-type-declaration) section and offers an illustrative example.

Student.h
```cpp
#ifndef STUDENT_INCLUDED
#define STUDENT_INCLUDED

//can't include Course here and Student in Course.h because that's a circular dependency

// compilers need to know how big the object will be
// this will be 10 times as big as a Course pointer
// but C++ already knows how big any pointer is (they're all 4 bytes)
// now you can just use an incomplete declaration to break circular dependency and speed up compilation 

class Course;

class Student
{
  public:
    void enroll(Course* cp);
  private:
    Course* m_studyList[10];
};
#endif // !STUDENT_INCLUDED
```

Course.h
```cpp
#ifndef COURSE_INCLUDED
#define COURSE_INCLUDED

class Student; // breaking circular dependency

class Course
{
  public:
  int units() const;
  private:
  Student* m_roster[1000];
};
#endif // !COURSE_INCLUDED
```

main.cpp
```cpp
#include "Student.h"
#include "Course.h"

void f(Student* s, Course* cp)
{
  s->enroll(cp);
}
```

## Incomplete Type Declaration

C++ allows for declarations of a class that don't give any other information. You can do this as many times as you like and still give a full declaration later.
```cpp
class A;
// ...
class A;
// ...
class A 
{
  //...
};
// ...
class A;
```
You can even keep giving incomplete declarations after the complete one!

You can use an incomplete declaration anytime you're just using a reference or pointer to the class type (C++ already knows the size). But if you have a data member of that type, you need to have the full declaration to see the full size.

In general:
> If the file Foo.h defines the class Foo, when does another file require you to say
> ```cpp
> #include "Foo.h"
> ```
> and when can you instead simply provide the incomplete type declaration
> ```cpp
> class Foo;
> ```
> 
> You have to `#include` the header file defining a class when
> * you declare a data member of that class type
> * you declare a container (e.g. an array or a vector) of objects of that class type
> * you create an object of that class type
> * you use a member of that class type

```cpp
class Blah
{
    //...
    void g(Foo f, Foo& fr, Foo* fp);  // just need to say   class Foo;
    //...
    Foo* m_fp;           // just need to say   class Foo;
    Foo* m_fpa[10];      // just need to say   class Foo;
    vector<Foo*> m_fpv;  // just need to say   class Foo;

    Foo m_f;             // must #include Foo.h
    Foo m_fa[10];        // must #include Foo.h
    vector<Foo> m_fv;    // must #include Foo.h
};

void Blah::g(Foo f, Foo& fr, Foo* fp)
{
    Foo f2(10, 20);      // must #include Foo.h
    f.gleep();           // must #include Foo.h
    fr.gleep();          // must #include Foo.h
    fp->gleep();         // must #include Foo.h
}
```

**As a result of these rules, you can't have a class B with a data member of class A when class A has a data member of class B.** There is infinite recursion here that you should avoid anyway (when the compiler goes to calculate object size it will get an infinite size).

## Resource Management

We don't manage infinite resources, so we'll focus on computer resources that are limited. We're going to focus on memory for this class.

Let's imagine that we're back in time. When c++ had no standard library. Let's try to write a standard String implementation.

```cpp
void h()
{
  String s("Hello");
  String t;

  char* p;
  // ... set p somehow
  String u(p);
}

class String 
{
  public:
    String(const char* value); //remember that any string literal is actually a cstring, and that any array is passed as a pointer to the first item
    String();
  private:
    // Class invariant:
    // m_text is a pointer to dynamically allocated array of m_len+1 characters
    // m_len >= 0
    // m_len == strlen(m_text)
    char* m_text; //dynamically allocate so we don't default to too much or too little memory
    int m_len;
}

String::String(const char* value)
{
  m_len = strlen(value);
  m_text = new char[m_len+1];
  strcpy(m_text, value);
}

String::String()
{
  m_len = 0;
  m_text = new char[1];
  m_text[0] = '\0';
}
```
Could you massage these into one constructor?
```cpp
String::String(const char* value = "")
{
  m_len = strlen(value);
  m_text = new char[m_len+1];
  strcpy(m_text, value);
}
// now you don't need separate default
```

We decide for empty string, should we just point to array of just null bit, or do we have the `nullptr`? There are time costs and memory costs for initializing null bits, but it's easier to implement. The needs of your users outweigh saving you time as a developer. So the right decision is `nullptr`.

Costly to calculate length, so perhaps we store an integer of length as a data member. 

Dealing with potential `nullptr`:

```cpp
String::String(const char* value = "")
{
  if (value == nullptr)
  {
    value = "";
  }
  m_len = strlen(value);
  m_text = new char[m_len+1];
  strcpy(m_text, value);
}
```

And we need a destructor, because the default constructor generated for built-in types does essentially nothing. We need to get rid of everything we dynamically allocated.

When you dynamically allocate a single object, you can just delete it. If you dynamically allocate an array of class objects, you use the array delete: `delete[]`. Must pass the pointer to element 0 with this command.

```cpp
String::~String()
{
  delete[] m_text;
}
```

When you have a local variable, the destructor is called automatically when you exit the bracket. If you have:

```cpp
struct Employee
{
  String name;
  double salary;
  int age;
};

void f(int age) 
{
  Employee e;
  e.salary = 12345.6;
  e.age = age;
}
```

The new Employee object we made gets cleaned nicely at the bottom because after the empty default destructor runs, each data member will get destructed by it's own specified or built-in destructor.

# Week 2 Monday Q&A

## Destructor Cycle

1. Run destructor body
2. Perform some action for each data member of an object.
   - If it's a class type, the class's destructor is called
   - If it's built-in, nothing happens
     - This is dangerous if it's a pointer to a dynamically allocated memory object. Because the pointer is built in, it won't do anything and the object in the heap will remain (memory leak).
3. Something else (not covered yet)

Note: objects can't contain a data member of a type that has a data member of the original type (infinite recursion in constructors). Smallberg says there's no issue if one of them is a pointer to the other object.

Don't have to dynamically allocate array for Project 1 because we are given upper bounds for dimensions.

# Week 2 Wednesday Lecture

More on resource management. We use our definition of `String` from last time.

## Copying

Consider the situation:

```cpp
void f(String t) {
  // ....
}

void h() {
  String s("Hello");
  f(s);
  // ... 
}
```

Here you pass by value. If you don't tell C++ what to do, it will default to just copying each member. So if `f()` changes the pointer member, we could affect the original object. Another problem, is that at the end of `f()`, all local variables get deleted, so your pointer in `s` is now dangling. Even if you don't touch `s` after, at the end of `h()` it will try to delete the object at the end of the pointer **again**, which is undefined behavior.

Let's make `t` have its own copy. We inform the compiler how to copy.

We add
```cpp
public:
  String(const String& other);
```
to the class declaration. And then we define it:
```cpp
String::String(const String& other) 
{
  m_len = other.m_len;
  m_text = new char[m_len + 1];
  strcpy(m_text, other.m_text);
}
```
This is called the "copy constructor," and C++ will use it by default when copying the object. It's illegal to pass by value to the copy constructor (infinite recursion). You are allowed to have your copy constructor modify the original object. 

Ways to make a copy:
```cpp
String x(s);
String x = s;
// and passing by value to a function
```

Note that you can talk about the private members of any String in the copy constructor (and any member function). They did this just to make it way easier to write copy constructors.

## Assignment

Some misconceptions:
```cpp
s= t; // assignment
String x(s); // copy construction
String x = s; // copy construction, NOT assignment
```

Consider:
```cpp
void h() {
  String s("Hello");
  f(s);
  String u("Wow");
  u = s; // how do we deal with assignment? 
}
```

The default is to just assign each data member. Pointer will point to the same place in memory as the original - which is a problem, and there will be a memory leak in the original value of `u`.

```cpp
u = s; // is shorthand for:
u.operator=(s);
```

Which we implement as:
```cpp
public:
  void operator=(const String& rhs); // not quite the current convention
```
in the class declaration. And then we define it:
```cpp
void String::operator=(const String& rhs) 
{
  delete m_text; // not quite right, we'll fix it later
  m_len = rhs.m_len;
  m_text = new char[m_len + 1];
  strcpy(m_text, rhs.m_text);
}
```

If we want this special behavior of built in types:
```cpp
int k = 3;
int n = 5;
int m;
m = k = n;
```
we modify our operator accordingly (return new value of LHS from an assignment statement).

```cpp
String String::operator=(const String& rhs) 
{
  delete m_text; // not quite right, we'll fix it later
  m_len = rhs.m_len;
  m_text = new char[m_len + 1];
  strcpy(m_text, rhs.m_text);
  return *this; // return what the pointer points to
  // but this implementation is a bit inefficient
}


// here's the fix:
String& String::operator=(const String& rhs) 
{
  delete m_text; // not quite right, we'll fix it later
  m_len = rhs.m_len;
  m_text = new char[m_len + 1];
  strcpy(m_text, rhs.m_text);
  return *this; // return a reference to original
}
```

But self-assignment could present a problem when you delete yourself before getting the value. Doing `strcpy` to itself is undefined behavior, probably keep reading and storing bytes until it gets to `\0`.

We'll revisit our function again:
```cpp
String& String::operator=(const String& rhs) 
{
  if (this != &rhs)
  {
    delete m_text;
    m_len = rhs.m_len;
    m_text = new char[m_len + 1];
    strcpy(m_text, rhs.m_text);
  }
  return *this; // return a reference to original
}
```
This is the classic way to write an assignment operator. But now there is a modern way. Note: our check for being the same is different than `*this != rhs`, because this compares the value of two strings. This is nice because it skips the work if it already has the right value, but it can be hard to check the equality of string value. Balance the cost between reassignment and checking if the value is the same. Generally we compare the pointers, not the values of the string.

Another note: technically you could replace the assignment operator definition with the destructor and copy constructor. But this is hard and uses stuff we don't know yet. The old way to avoid code duplication was to write a private helper function.

Here's another problem: in our current implementation of the assignment operator, we do serious damage to the left-hand side before we know that our operation will succeed. This can lead to undefined behavior later in the program. The following fix is just a cleaner, modern way to write assignment operators.
```cpp
class String
{
public:
  //...
  void swap(String& other);
  //...

private:
  //...
};

void String::swap(String& other)
{
  // simple operations like this can't throw exceptions
  // I decided to implement this more concretely
  int temp_len = m_len;
  m_len = other.m_len
  other.m_len = temp_len;
  char* temp_textptr = m_text;
  m_text = other.m_text;
  other.m_text = temp_textptr;
  // and then local temp variables are cleaned up
}

String& String::operator=(const String& rhs) 
{
  if (this != &rhs)
  {
    String temp(rhs); // will fail here if there isn't memory
    swap(temp); // so lhs is protected
    // note that the above is this->swap(temp);
  }
  return *this; // return a reference to original
}
```
The above solution also leverages the code written in the copy constructor to prevent code duplication.

## Aliasing

An "alias" is two names for the same object. Ask yourself: is my code correct if these names refer to the same value?

Example:
```cpp
void transfer(Account& from, Account& to, double amt)
{
  if (amt > from.balance())
  {
    //... error
  } else
  {
    from.debit(amt);
    to.credit(amt);
  }
}
```
Here there is an issue of aliasing. When you have two references to the same type, aliasing might occur. Nothing catastrophic happens here, but it treats a self-to-self transfer like a standard transfer, so it will throw errors, incur fees, etc. Make sure your implementation does the right thing in the case of aliasing.
```cpp
void transfer(Account& from, Account& to, double amt)
{
  if (&from != &to)
  {
    if (amt > from.balance())
    {
      //... error
    } else
    {
      from.debit(amt);
      to.credit(amt);
    }
  }
}
```
# Week 2 Wednesday Q&A

- When a function runs but can't do its job, the best thing it can do is return without changing anything. This is motivation for our `swap()` approach to assignment operator.
- Sometimes copying a resource needs more thought than just copy or don't copy. You can address this in your override (think about a network connection).

I watched this lecture to 13:15.
