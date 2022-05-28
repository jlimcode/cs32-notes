# Week 5 Monday Lecture

## Motivating Recursion

When you have a big problem, you can solve it by breaking it into smaller problems. When the smaller problems are just smaller versions of the big one, then you can use recursion.

When you have a lower bound for the size of the job, and each recursive call works with a strictly smaller problem, then the recursive algorithm must terminate. (Sounds like Monotonic Convergence Theorem to me).

## Introduction to Recursion

base case(s): path(s) through the function that do not make a recursive call
recursive case(s): path(s) through the function that make a recursive call

Every recursive call is to solve a strictly "smaller" problem (getting closer to base case)

You must make the "recursive leap of faith" when you know the base case works, and you assume the function will work for smaller instances of the same problem and just make the recursive step get closer to the base.

## Implementing the Merge Sort

Turns out it's easier to represent an array segment with the start and the length, rather than the start and the end (end leads to lots of +1 and -1). Then if `b` is your beginning, `e` can be the index right past the end, then `m = b + e / 2` is the point right past the end position of the first segment.

```cpp
void sort(int a[], int b, int e)
{
    if (e - b >= 2)
    {
        int mid = (b + e) / 2;
        sort(a, b, mid);
        sort(a, mid, e);
        merge(a, b, mid, e);
    }
}
```

# Week 5 Wednesday Lecture

## Pointers and Arrays

**Important note on array parameters!** Pointers are not the same as arrays. However, when you declare a function with an array parameter, all c++ knows is that it's a pointer to the array type. You could pass `f(int[] a, int n)` as `f(int* a, int n)` and it would work the same. When you pass it in, it's just a pointer so **you cannot possibly know what the size of the array is**. Thus, you need a size parameter that just gets treated as fact.

## Infinite Recursion

If you accidentally set up an infinite recursion, your program will eventually crash when you run out of allocated space for local variables. This is different than an infinite loop where no resources are being consumed other than power.

## Types of Recursive Problems

Divide and conquer or the first and the rest or the last and the rest.

Divide and conquer works well for sorting because you need to view each item more than once. Whereas the other two work for more types of problems like we're solving in homework 3.

## Common Mistakes in Recursion

We can think of these in mistakes in a proof by induction. Did you do the base case or inductive step incorrectly?

Sometimes your inductive step fails for a small $n$ like 1 or 2. Can come up when you're inferring about a larger collection based on the smaller collection.

An example mistake:

```cpp
bool has(int a[], int n, int target)
{
    if (a[0] == target)
    {
        return true;
    }
    return has(a+1, n-1, target);
}
```

This won't work, but clearly something is wrong because there's no place it can return `false`. The only base case is when you match the element, so this seems off as well. Also it looks like the recursive calls all decrease `n`, but there doesn't appear to be a lower bound in the function. Here's the fix:

```cpp
bool has(int a[], int n, int target)
{
    if (n <= 0)
    {
        return false;
    }
    if (a[0] == target)
    {
        return true;
    }
    return has(a+1, n-1, target);
}
```

## Templates

We're back on a computer language type topic. A way to make your code writing more efficient if you have a language that can support this feature.

Here's the type of problem we're trying to solve:

```cpp
int minimum(int a, int b)
{
    if (a < b)
        return a;
    else
        return b;
}

double minimum(double a, double b)
{
    if (a < b)
        return a;
    else
        return b;
}
```

These two implement the same algorithm for different types. Note that the underlying machine code will actually be different. We're just telling the compiler how to make machine code matching the **template** of this algorithm.

New keyword: `template`

```cpp
template<typename T>
T minimum(T a, T b)
{
    if (a < b)
        return a;
    else
        return b;
}
```

The compiler generates everything at compile time, doesn't consider the template as a function until it is called with typed objects. When the template is called the first time, the compiler generates the code and machine code for the right function. Then this code can be reused in subsequent call. 

When the template is called with some objects, it generates the code through a process called **template argument deduction**.
1. The call matches some template
2. The instantiated template must compile
3. The instantiated template must work as intended

In template argument deduction, the compiler doesn't consider possible conversions like `double` to `int`. All the operators and functions called in the template must be defined for the type passed in. This will also result in a compiler error.

Here's an example where the instantiated template doesn't work as intended.

```cpp
int main(int argc, const char** argv) {
    char ca1[100];
    char ca2[100];
    cin.getline(ca1, 100);
    cin.getline(ca2, 100);
    char * ca3 = minimum(ca1, ca2);
    return 0;
}
```

This will just compare the two `char` pointers. But thankfully, the compiler will match normal functions defined for the type over templates, so we can define a special version of `minimum` for `cstring`s.

For matching, the only conversions considered are:
- `A` to `A&` 
- `A` to `const A`
- array of `A` to `A*` 

Note that matching is only performed based on the parameter types, not the return type.
