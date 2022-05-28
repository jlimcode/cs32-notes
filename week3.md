# Week 3 Monday Lecture

## Introduction to Linked Lists

So far all we've seen in terms of data types are arrays (fixed size or dynamically allocated). Technically dynamically allocated is called "dynamically allocate fixed size array" because the size can't be changed after creation.

We can create a "resizable array" by redefining our insert function to allocate more storage if the existing array runs out of room. Called a "vector" in the c++ standard library.

In an array, it's computationally cheap to find an item, or put a new item at the end. Inserting is more expensive because you have to move everything over (the closer to the beginning you insert, the more costly it is). You would need to insert when you want to keep the items in a particular ordering.

Goal: represent a collection of values for which inserting and removing items preserves the order of the other items, but is efficient even if the insertions are not at the end.

What if we make the elements of our collection **not** right after each other in memory. For each element, you need the data item and where the next one is. This structure is called a link list.

## First Link List Implementation

```cpp
struct Node
{
    int value;
    Node* next; 
    // must be a pointer to avoid recursive size equation
};

Node* head;
```

## Linked List Iteration

Let's assume we already have the linked list set up. Let's write code that tries to visit each node and print.

```cpp
while(head != nullptr) 
{
    std::cout << head->value << std::endl;
    head = head->next;
}   
// MAJOR MEMORY LEAK!!
```

We've cut ourselves off from all the nodes in our program. We shouldn't change the `head` pointer!

```cpp
for(Node* p = head; p != nullptr; p = p->next)
    std::cout << p->value << std::endl;
// this one works
```

General rule: whenever you write `p->_____`,
- make sure p is not uninitialized
- make sure p is not nullptr

So for this, make sure you always initialize `head` and all values of `next` with valid pointers.

## Finding a Specific value

Finding first occurrence of 18 in our linked list.

```cpp
Node* p;
for(p = head; p->value != 18; p=p->next)
    ; //in c++ this is the empty statement, which does nothing
```

This doesn't work because it assumes an 18 will be found. It also doesn't check for p being the nullptr. p could be nullptr from either head or p->next

```cpp
Node* p;
for(p = head; p != nullptr && p->value != 18; p=p->next)
    ; //note the order of conditions to avoid evaluation with nullptr
```

Insert a 54 after the 18 in the list, if present:

```cpp
Node* p;
for(p = head; p != nullptr && p->value != 18; p=p->next)
    ;

if(p != nullptr) 
{
    Node* newGuy = new Node;
    newGuy->value = 54;
    newGuy->next = p->next;
    p->next = newGuy;
}
```

Note on order: can't go wrong if you set the fields of the newly allocated node first.

## Getting Rid of a Node

Get rid of the node after the 18 node:

```cpp
Node* p;
for(p = head; p != nullptr && p->value != 18; p=p->next)
    ;

if(p != nullptr) 
{
    Node* toBeDeleted = p->next;
    p->next = p->next->next; // or toBeDeleted->next;
    delete toBeDeleted;
}
```

Get rid of the first occurrence of 18 (somewhat trickier because you can't trace a pointer backwards):

You could do it in two loops by finding 18 and then the node pointing to 18. Or you can do it in one loop with a lagging pointer that follows the node you're checking.


Tips:
- check how the list works when the area of interest is
  - at the front
  - at the back
  - in the middle
- check one-element list

What if our 18 is at the end of the list, and we want to insert 54 after?

```cpp
if(p != nullptr) 
{
    Node* newGuy = new Node;
    newGuy->value = 54;
    newGuy->next = p->next; //nullptr
    p->next = newGuy;
}
```

Here, our previous implementation works. However, if we're trying to delete a specific item from the very front of the list, the general algorithm doesn't work. We rewrite to handle this special case:

```cpp
head = p->next;
delete p;
```

If you're doing a lot at the tail of the list, you might want to keep a pointer to the tail.

Adding an item to the end is simple:

```cpp
Node* newEnd = new Node;
newEnd->value = 87;
newEnd->next = nullptr;
tail->next = newEnd;
tail = newEnd;
```

Empty list: both head and tail will be nullptr, so this is a special case.

You could also address the "deleting a specific node" problem by storing two pointers in each node, one pointing to the previous node and one pointing to the next node. This is called a **doubly-linked list**.

# Week 3 Wednesday Lecture

Data structure where you can only add or remove items at one end of the structure is called a **stack**. Using this structures tells your reader exactly what you're using the list-type structure for. You can push to the end to add, pop from the end to remove. You can also look at the top item. And check if the stack is empty. Some libraries will give you functions to look at any member and check how many items there are. 

## Stacks in c++

In c++ you use `#include <stack>` to get `std::stack` in your code. Looking at the top of an empty stack or trying to pop from an empty stack is undefined behavior in c++.

```cpp
#include <stack>
using namespace std;

stack<int> s;
s.push(10);
s.push(20);
cout << s.top() << endl;
s.pop() // no return
if (s.empty())
{
   cout << "Stack is empty!" << endl; 
} else
{
    cout << s.top() << endl;
    cout << s.size() << endl;
}
```

## Queues in c++

If we look at a similar data type with one side where you add new elements and one side where you remove elements. This is called a queue. The add side is called the tail or the back, the remove/view side is called the head or front.

In general, we have the operations enqueue to add to back of queue, dequeue to remove from the front of the queue, look at the front end of the queue, and check if the queue is empty. Some libraries will let you view the back item, view any item, and ask how many items are in the queue.

```cpp
#include <queue>
using namespace std;

queue<int> q;
q.push(10);
q.push(20);
cout << q.front() << endl;
q.pop() // no return
if (q.empty())
{
   cout << "Queue is empty!" << endl; 
} else
{
    cout << q.size() << endl;
    cout << q.back() << endl;
}
```

Once again trying to view the front, back or pop from an empty queue is undefined behavior.

## Motivating Example for Stacks

Prefix notation: $f(x,y,x)$ or `add(sub(8, div(6,2)), 1)`

Infix notation: this is our standard mathematical syntax $8-6/2+1$ but it doesn't give the full context so we must have rules of precedence.

Postfix notation: 6 6 2 / - 1 + which is once again unambiguous. 

In calculator history, Texas Instruments created a calculator that you could enter your calculations in standard infix notation. But this took up code on the chip so they couldn't add other functionality beyond square root. HP took a different approach and made a calculator that could do many different calculations, but you had to enter your calculations in postfix notation.

Calculating postfix with a stack:

1. If it's a number, put it on the stack
2. If it's an operator, pop off the number of operands off the stack. Then calculate and push the result back onto the stack

Converting infix to postfix with a stack:

1. If you get an operand, append it to the result sequence
2. Else if the current it is (, push it onto the stack
3. Else if the current item is ), pop operators off the stack, appending them to the result sequence until you pop at (, which you don't append
4. If the current item is an operator:
   - If the current stack is empty, push the current operator onto the stack
   - Else if the top of the stack is (, push the current operator onto the top of the stack
   - Else if the current operator has precedence strictly greater than the operator on the top of the stack, push the current operator unto the stack
   - If it doesn't have higher precedence, pop the top operator from the stack and append it to the result sequence
     - Then check again and repeat if necessary
5. At the end of the input sequence, pop each operator off the stack and append it to the result sequence.

## Implementing Stacks with Arrays/Lists

Using a pointer for array item. Points to next available space. Be careful not to pop or view top from an empty stack. 

Linked list will start at the top with a single linked list, where it will add and pop from. Don't pop or view top from an empty stack.