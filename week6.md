# Week 6 Monday Lecture

## More issues with Templates

You don't know if your client will call your function with something that is expensive to copy. So if you don't need to change it, just pass by constant reference in your template.

```cpp
template<typename T>
T minimum(const T& a, const T& b)
{
    if (a < b)
        return a;
    else
        return b;
}
```

And sometimes you run into some other problems like

```cpp
template<typename T>
T sum(const T& a[], int n)
{
    T total = 0;
    for (size_t i = 0; i < n; i++)
    {
        total += a[k];
    }
    return total;
}
```

This works really nicely for doubles and ints, but not strings because the initial declaration doesn't work. You can't say `string total = 0;`

Can't use default constructor, works for strings but built in types don't have default constructors. But the c++ people realized that there was no issue if they made it allowed for built-in types. So now `double()` can be called to get 0. Same exists for `char`, `bool`, etc. You get something analogous to 0.

## Template Classes

If you write a class that you want to work for many types, and all you would have to do to re-implement it is change some type names, then you can use a template.

```cpp
template<typename T>
class Stack
{
    public:
        Stack();
        void push(const T& x);
        void pop();
        T top() const;
        int size() const;
    private:
        T m_data[100];
        int m_top;
};
```

Then you put `template<typename T>` over every member function definition.

```cpp
template<typename T>
void Stack<T>::Stack() : m_top(0)
{}
```

You put `T` in the classname too (the alias name you choose for type).

You only instantiate the member functions you use. So you can have a template that wouldn't have all the functions work, you just can't use any of the ones that won't work for that particular type.

ex. you can push a `double` to an `int` stack. Because we're done matching at this point, we know what type `T` ought to be, so we are allowed to pass `double`s to this `int` function.

## STL

The c++ people decided that some of the library things needed to be standardized. Someone had already started to create some standard libraries. But HP engineers created their own standard libraries called STL for Standard Template Library. It was so clever that c++ decided to make it the standard. STL includes all of the data structures we've seen so far.

## Vector

```cpp
#include vector
using namespace std;

vector<int> vi;
vi.push_back(10);
vi.push_back(20);
vi.push_back(30);

vi.size();
vi.front();
vi.back();

for (size_t i = 0; i < vi.size(); i++)
{
    cout << vi[i]; // overloaded square brackets
}

vi.pop_back();

vi.at(1) =  60;
vi.at(3) =  70; //throws an exception (bounds checking)

vector<double> vd(10);
// vd.size() is 10, each element is 0.0

vector<string> vs(10, "Hello");
// vs.size() is 10, each element is "Hello"

int a[5] = {10, 20, 30, 40, 50};
vector<int> vx(a, a+5);
// vx.size() is 5, each element is a copy of corresponding in `a`
// you pass a pointer to first and pointer right past last element
```

Common mistake:
```cpp
vi[0] = 10; //undefined!!, might not be allocated
vi.push_back(10); //correct
```

Note: clang and g++ use size and capacity as integers. Microsoft with Visual c++ uses pointers. This small difference would really only come up in a debugger because they work the same.

Note: as you add objects to the vector, the items can move because we need to make new storage that's bigger. So if you're working with pointers, this could be problematic. This concept is called "invalidation." Don't traverse a vector with pointers while you're modifying it.

## Linked Lists

Note: In general, STL will only implement a method if it will be efficient

```cpp
#include <list>
using namespace std;

list<int> li;
li.push_back(20);
li.push_front(10);

li.size();
li.front();
li.back();

li.push_front(40);
li.pop_front();

//no square bracket operator!

li.begin(); //iterator to beginning
li.end(); //iterator pointing to right past end

//if list is empty, li.end() == li.begin()

//STL doesn't give you nodes or let you look at them
//instead you get something like a pointer, but an iterator

for (list<int>::iterator = li.begin(); p != li.end(); p++)
{
    // ^^ nested type (must be public)

    cout << *p << endl;
}
```

Iterators overloaded some syntax like `*p` to get the value and `p++` moves to the next element. 

Another fun thing (copy from lists to vectors):

```cpp
list<string> vs(ls.begin(), ls.end());
```

You can also do:

```cpp
list<int>::iterator p = li.end();
p--;
p--;
//p-=2 doesn't compile
```

Now you're looking at the 2nd to last element.

Then:

```cpp
//inserts at our location p
list<int>::iterator q = li.insert(p, 40);
//q will point to location of 40, p will point right after it
```

You can also erase:

```cpp
list<int>::iterator q = li.erase(p);
//it's now undefined to do anything with p
//q is now a pointer to the element right after p, you could assign this back to p
```

We also have iterator types for any container type. These have more pointer arithmetic.

```cpp
p += 2;
//will be defined for vector::iterators
```

Vector iterators do have `insert(p, val)` functions, even though it could be inefficient at the beginning of the vector. This insert still returns a new pointer to the place the value was inserted. This is necessary because your items could be in a new place after allocating more space.

So typically you do:
```cpp
p = vi.insert(p, 40);
```
After you erase at a point, it's undefined to do anything to that iterator. This follows the standard of lists and also you don't know what your pointer will point to, so might as well not exist.


# Week 6 Wednesday: STL and Algorithms

## Standard Algorithms

We've looked at data types in STL, but there are also algorithms associated with those libraries. The convention in STL is to pass an array as a pointer to the first element and a pointer right past the last element. We return a pointer to the item of interest. If the pointer returned is equal to the pointer just after the last element.

We'll start with
```cpp
int* find(int* b, int* e, const int& target)
{
    for(; b!= e; b++)
    {
        if (*b == target)
            break;     
    }
    return b;
}
```

First we make it more general by making a template:

```cpp
template<typename Iter, typename T>
Iter find(Iter b, Iter e, const T& target)
{
    for(; b!= e; b++)
    {
        if (*b == target)
            break;     
    }
    return b;
}
```

Now it will work with arrays, lists, vectors, integers/cstrings etc. This is very similar to the actual `find()` in STL.

STL algorithms embody very simple loops, mostly just for readability and standardization. Follow a consistent set of conventions that c++ programers are familiar with. You access this with `#include <algorithm>`.

You often find that you need to do something similar to the STL function but a little different. You shouldn't rewrite these. You can pass your own functions to STL to perform more custom operations. Your own functions need to have the right arguments and return types to work. You can also do this with sort.

```cpp
bool hasBetterRecord(const Team& t1, const Team& t2)
{
    if (t1.wins() > t2.wins())
        return true;
    if (t1.wins() > t2.wins())
        return false;
    return t1.ties() > t2.ties();
}

int main()
{
    std::vector<Team> league;
    std::sort(league.begin(), league.end(), hasBetterRecord);
}
```

That's it for STL right now.

## Algorithm Analysis

When algorithms were first being shared, there was no good way to compare the performance between algorithms because the speed of each computer is different. Instead we count all of the basic steps that the algorithm requires. 

Note: for large $N$, the smaller terms of the polynomial become irrelevant. We don't care that much about performance of small $N$ because a computer can do those really quickly anyway. To simplify further, we're just looking at the overall growth behavior i.e. the highest exponent of $N$.

Theorem: A function $f(N)$ is $O(g(n))$ if there exists $N_0$ and $k$ such that for all $N\geq N_0$, $|f(N)| \leq k\cdot g(N)$.

Example:
```cpp
for (int i = 0; i < N; i++) // <=========== O(1) * N = O(N)
{
   c[i] = a[i] * b[i]; // <========= O(1) 
}

for (int i = 0; i < N; i++) // <===========  O(N^2)
{
    a[i] *= 2;
    for (int j = 0; j < N; j++) // <===== O(N)
    {
        d[i][j] = a[i] * c[j];
    }
}

for (int i = 0; i < N; i++) // <=========== O(1+2+....+(N-1)) = O(N^2)
{
    a[i] *= 2;
    for (int j = 0; j < i; j++) // <===== O(i)
    {
        d[i][j] = a[i] * c[j];
    }
}
```

The last two are treated basically the same even though the second is twice as fast as the first. We ignore constant of proportionality for this level of analysis.

Here's a comparison of our growth rates.

<iframe src="https://www.desmos.com/calculator/2xewy5bxld?embed" width="500" height="500" style="border: 1px solid #ccc" frameborder=0></iframe>

The classic method of matrix multiplication has a high polynomial growth rate. You can also have $O(2^N)$ algorithms which are really bad. This of generating all the possible sets of items in a set.