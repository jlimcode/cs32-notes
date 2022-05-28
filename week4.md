# Week 4 Monday Lecture

## Implementing Queue with an Array

Your head and tail are two pointers that indicate the front and end of the queue in the array. Popping from the front moves the head back and enqueueing to the back adds elements and shifts the tail pointer. 

This continues until you attempt to enqueue and reach the end of your array. Do you give up? No because there could be spots in the array freed by popping. Do you shift everything over to give more space for enqueueing at the end of the array? No because this is costly copying.

So the clever idea is to wrap around to the beginning and put the 101th element in item 0's spot. Then when you pop past the 100th element, you move it back to the beginning. Then you can wrap around and easily use all 100 spaces in your array for the queue. We're treating linear array memory like it's a circle. This can be called ring buffer or circular array.

Putting the tail pointer one past the last item in the queue, you get `tail == head` when the queue is empty **or** if the queue is full. Then you should probably keep the size so you know it's empty if `size == 0`, and full if `size==100`. If you have an expandable array, make sure you copy head into 0, tail into 100 rather than their original indexes because those won't wrap correctly.

## Polymorphism

This is a general programming language concept rather than a c++ specific concept or a data structure.

In c++ you can put objects of different types into a collection, but you have to be careful about it so your program can check stuff at compile time.

Note:
```cpp
Circle* ca[100];
// create an array of pointers to circles
// this avoids creating 100 circles along
// with the array
```

Polymorphic implementation

```cpp
class Shape
{
    //base class
    public:
        void move(double xnew, double ynew);
        void draw() const;
    private:
        double m_x;
        double m_y;
    // these properties shared by all shapes
};

class Circle : public Shape
{
    //derived class
    public:
        void move(double xnew, double ynew);
        void draw() const;
    private:
        double m_x;
        double m_y;
        double m_r;
};

class Rectangle : public Shape
{
    //derived class
    public:
        void move(double xnew, double ynew);
        void draw() const;
    private:
        double m_x;
        double m_y;
        double m_dx;
        double m_dy;
};

Shape* pic[100];
pic[0] = new Circle;
pic[1] = new Rectangle;
```

Every object of a derived type has an object of the base type embedded inside it. So it will have everything a `Shape` object plus whatever else you add. If you give a pointer to a `Circle` but you need a pointer to `Shape`, the compiler will do an automatic conversion to allow you to get the right kind of pointer.

Compilers optimize this conversion by putting the start of the base object right up at the top of the derived object. That way to "convert" between pointers you simply add 0 (just return the pointer as is).

```cpp
for (size_t i = 0; i < MAX_SIZE; i++)
{
    pic[i].draw();
    //this works now through inheritance
}
```

## Implementation of Polymorphism

```cpp
Shape::move(double xnew, double ynew)
{
    m_x = xnew;
    y_x = ynew;
}
// this will work for all shapes
```

Drawing will require different functions for the different kinds of shapes. Because you added these function declarations to the class declarations of the derived types, the compiler will look for their separate definitions.

```cpp
void Circle::draw() const
{
    //draw a circle with m_r at x,y
}
```

This is called an override for the `draw()` function.

But because you gave Shape a draw method too, you need to define it. But what makes sense to draw? We'll implement now and fix it later.

```cpp
void Shape::draw() const
{
    //draw some vague cloud at x,y
}
```

## Static and Dynamic Binding

Static binding vs. dynamic binding: putting the code together with the function call at compile time vs. putting the function body to the call at runtime. We cannot statically bind `draw()` in this example. Other languages will always just dynamically bind, but c++ wanted the performance boost of statically binding by choice. The default is actually static binding, so you need to indicate if you want dynamic. 

The consequences for our program are that `move()` and `draw()` will always call `Shape`'s version because they have been statically bound during compile. This is fine for `move()` but not for `draw()`. Here's how we indicate that we want dynamic.

```cpp
class Shape
{
    //base class
    public:
        void move(double xnew, double ynew);
        virtual void draw() const;
        // don't have to repeat is subclasses, but
        // you can just to help readability
};
```

If you let a method be statically bound, then you don't have to write an override in subclass because it will never get called. But here you're making a prediction about the future that you're never going to want to override the method in any future subclasses.

```cpp
class WarningSymbol : public Shape
{
    void move(double xnew, double ynew);
};

void WarningSymbol::move(double xnew, double ynew)
{
    Shape::move(xnew, ynew);
    // call the correct move function
    // then do something else
    // like flashing
}

void f(Shape& x)
{
    x.move(10, 20);
}

WarningSymbol ws;
ws.move(5, 15); //this will work
f(ws); //this calls the wrong move
       //because statically bound 
```

So when you want to override, you have to specify `virtual`.

# Week 4 Wednesday

## Dynamic Lookup

Some driver code to illustrate how the compiler matches the right function bodies with a seemingly ambiguous function call/

```cpp
Shape* sp;
if (someCondition)
{
    sp = new Shape;
} else
{
  sp = new Rectangle;  
}
sp->draw();
```

How do we call the right draw?

The C (naive-ish) way: include an integer code in Shape that specifies what type of shape it is. Then when you call `draw()`, the function checks the integer code of the object and calls the correct version of itself. This is bad because it requires complete recompilation of everything involving Shape every time you want to add a new class that implements Shape. There will need to be a new integer code and all function calls to `draw()` have to recognize it. This recompilation is costly and should be avoided. 

The c++ way: compiler sets up a table for Shape called a virtual table of `vtbl`. Has an entry for each virtual function of the `Shape` type. Tells the linker to put pointers to Shape's draw and move functions in the table. Rectangle will also have a virtual table for every virtual function in the class (move, draw, diag). The one's already declared in Shape will be in the same slots in Rectangle's virtual table as in Shape's. Diag will have a pointer to Rectangle's function. Draw will point to **Rectangle's** draw() function, and the move will point to **Shape's** move function. Compiler knows draw() is in slot #1 (arbitrary) for every class. Compiler gives an extra pointer called the virtual pointer for any object with virtual methods, `vfptr` in VS. In the constructor for rectangle, make a pointer in the Shape "subset" object to Rectangle's virtual table. So the machine code is
```c
"call the function at"
sp->vptr[1]; // the function in the second slot of 
             //whatever virtual table you have
```

Note that this says nothing about specific derived types. This gives us the flexibility to introduce new derived types of Shape without recompiling our code.

## Abstract Classes

Our implementation of Shape's draw function is kinda silly. Do we need it? Right now, yes because the compiler needs to guarantee that every Shape pointer object will lead to something drawable. But if I say that I'm not going to specify a draw function for Shape but that every derived class will have one, that may work. You want to set the pointer in the lookup table to `nullptr`.

Here's how you do it:
```cpp
class Shape
{
    virtual void move(double xnew, double ynew);
    virtual void draw() const = 0; //no new keywords lol
    // called a "pure virtual function"
    double m_x;
    double m_y;
}
```

Consequences: you can't make objects of the Shape type. So none of these:

```cpp
Shape* sp = new Shape;
Shape s;
```

But pointers to Shape with other things in there are fine. Then Shape is called an *abstract base class*, sometimes called ABC. 

If you don't declare a draw function in a subclass, you won't be allowed to instantiate that class either.

## Multiple Inheritance?

```cpp
class Shape
{
    //...
};

class Polygon: public Shape
{
    ~Polygon();
    Node* head;
}
```

Shape just had `int`s, so we didn't need a destructor, but Polygon is a linked list so we need to clean it up.

```cpp
Shape* sp;
if (someCondition)
{
    sp = new Polygon;
}
else
{
    sp = new OtherShape;
}

delete sp; //which destructor to call?
```

You need your destructor to be virtual, but Shape doesn't declare a destructor anywhere. And we can't default to virtual for every class, because that's slow. So we declare Shape's destructor. So

```cpp
class Shape
{
    //...
    virtual ~Shape();
};

class Polygon: public Shape
{
    virtual ~Polygon();
    Node* head;
}
```

But what if Shape is abstract, then there's never a Shape object to destroy. But turns out you still need that Shape destructor. Recall the destructor cycle:

1. Execute the body of the constructor
2. Destroy the data members
3. Destroy the base part

So deleting a Polygon will call Shape's destructor. So we have to implement it. Even if all our subclasses don't have destructors, the Shape destructor still gets called. So we give the somewhat lame definition:

```cpp
Shape::~Shape()
{}
```

So just write destructors for anything that may be a base class. Avoids recompilation later.

Now let's review the constructor cycle. It's the destructor in reverse!

Construction:
1. Construct the base part
2. Construct the data members
3. Execute the body of the constructor

Destruction:
1. Execute the body of the constructor
2. Destroy the data members
3. Destroy the base part

Here's our setup:

```cpp
class Shape
{
    public:
        Shape(double x, double y);
        virtual ~Shape();
    private:
        double m_x;
        double m_y;
};

Shape::Shape(double x, double y)
    : m_x(x), m_y(y)
{}

class Circle : public Shape
{
    public:
        Circle(double r, double x, double y);
        Circle(double r);
    private:
     double m_r;
}
```

The following code doesn't compile:

```cpp
Circle::Circle(double r, double x, double y)
    : m_x(x), m_y(y), m_r(r)
{}
```

Because we're trying to access `m_x` and `m_y` which aren't members that Circle can access, they're private to Shape. So we should initialize those with Shape. (First step of construction is construct the base part).

```cpp
Circle::Circle(double r, double x, double y)
    : Shape(x, y), m_r(r)
{}
```

An additional piece of information is given to the Shape constructor that points to Circle's virtual table. This is how it knows it's a Circle.

What if we have...

```cpp
Circle::Circle(double r, double x, double y)
    : m_r(r)
{
    if (r<=0)
    {
        //error handling
    }
}
```

Then the compiler looks for a default constructor for Shape, but since there is none this is a compilation error.

And our other Circle constructor looks like:

```cpp
Circle::Circle(double r)
    : Shape(0, 0), m_r(r)
{
    if (r<=0)
    {
        //error handling
    }
}
}
```