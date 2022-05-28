# Week 9 Monday Lecture: More Hashtables

A hash function must:
- produce uniformly distributed values
- cheap
- same key gets same value every time (deterministic)

What's the big O? A hash table with a fixed number of buckets has $O(N)$ for insertion, deletion, finding. So this performs worse than a BST. However, the rate of linear growth is really slow, so $N$ must be really large before it gets worse than $O(\log N)$.

To beat BST, we assume some constant load factor that we never want to exceed (a common choice is 0.75). We increase the number of buckets when an item about to be inserted would put us over the max load factor. Generally, then you make twice as many buckets. Rehash each item and put them into the new range. With fixed maximum load factor, looking things up is a constant time $O(1)$. Insertions will **mostly** be constant time. Every so often, rehashing is $O(N)$, but this happens rarely and it become rarer and rarer. Over time, this makes insertion constant on average. We can make a small improvement by not rehashing all at once, and generating a new table as we insert. This is called **incremental rehashing**. Bounded by a lower constant. Area under the curve is the same.

## More STL Data Structures

Last time we looked at `set` and `multiset`, which both use some sort of Binary Tree to do fairly fast lookup. Now we look at `map`.

```cpp
#include <map>
#include <string>

map<string, double> ious;

using namespace std;

string name;
double amt;

while (cin >> name >> amt)
{
    ious[name] += amt;
}

for(map<string, double>::iterator p = ious.begin(); p != ious.end(); p++)
    cout << p->first << " owes me \$" << p->second << endl;
```

Uses a `std` type called `pair` with a public data member `first` and `second` that are of each specified type.

Set types have the requirement that you iterate through their elements in increasing order. This doesn't work with a hash table.

There are types called `unordered_set` which doesn't require less than operator, just equality. Then there's no particular order. Also, `unordered_multiset`, `unordered_map`, and `unordered_multimap`.

# Week 9 Wednesday Lecture: Heaps

This is a data structure where you assign a priority to each item as you put it into a collection, then when you pull out an item you get the highest priority item. A stack or a queue are just specific examples of this structure. 

There are different conventions about whether high or low numbers have the highest priorities. We'll use high numbers to mean high priority in this class.

A good data structure for finding elements of highest priority is a BST. Both insertion and removal are $O(\log N)$. Because we always pull from the right most side, we unbalance our tree. So we can do better. 

A **complete binary tree** is filled at ever level, except possibly the deepest level, which is filled from left to right. There is only one unique complete tree for each number of nodes.

A **(max) heap** is a complete binary tree in which the value at every node is greater than the values of all the nodes in its subtrees.

A **min heap** is a complete binary tree in which the value at every node is less than the values of all the nodes in its subtrees.

There's no requirement that left and right children have any relation to each other.

When you remove from the heap, you take the root away. The algorithm for restoring the heap property has two parts:
1. Remake the binary tree
2. Restore the heap property

To remake the binary tree, you promote a node at the bottom right to the root. Then you "trickle down" the lower value by swapping parents with their max child until the parent is greater than both their children.

Thus, removing an element is worst case $O(\log N)$ (with a better constant of proportionality than a binary tree).

Inserting an element:
1. Add to the right spot to make it a complete binary tree
2. Then compare it to its parent and bubble it up until it's smaller.

Now here's a structure that can find its bottom right element in constant time. You can represent the heap as an array if you go through amd number each element of the tree by level left to right. How do you preserve parent-child relations? There's a function that maps child to parent indexes. 

$$\operatorname{parent}(i) = \left\lfloor \frac{i-1}{2} \right\rfloor$$

We can also find a function for children: 

$$\operatorname{children}(i) = 2j+1, 2j+2 \quad \text{(if they're in the tree)}$$

Now we don't have a tree, we can do everything with the array.

You can also sort items by putting them all into a heap and pull them out one by one. This is called **heapsort** and it has $O(\log N!) \approx O(N\log N)$.

Heapsort algorithm:
1. Make the array into a heap
2. Repeatedly remove items from the heap

For step 1, work you way backwards, making everything into a heap. Trickling up with a similar algorithm to how you removed items. Step 1 is the complexity $O(N\log N)$. Although we've put the large values in the front of the array, we're still sorting in increasing order. Now you swap across an imaginary barrier where everything to the right is sorted. Your heap shrinks in the left part of the array. You use the standard removal algorithm. When you're down to 2 items, just swap them if they're in the wrong order, otherwise you're done. This step is also $O(N\log N)$. Quicksort still has a better constant of proportionality on average, but a bad worst case. Typically introsort uses quicksort and then heapsort if it gets too deep. 