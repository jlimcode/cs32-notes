# Week 8 Monday: Trees

We were working with this structure this time:

```cpp
#include <vector>
#include <string>

struct Node
{
    std::string name;
    std::vector<Node*> children;
};

Node* root;
```

What we did to print before was called a preorder traversal. You process the node before processing the subtrees. The opposite is postorder traversal must process all the subtrees before processing the current node. If we did our printing in postorder, we would get the same output in the opposite order.

Early in CS, people didn't really like to use dynamically allocated arrays for children pointers. You can represent every tree by nodes with exactly two possible children, called a **binary tree**.

Let's look at the data structure:

```cpp
struct Node
{
    string name;
    Node* right;
    Node* left;
};
```

Note that the right and left subtrees are distinct. So there are 3 possible distinct two node trees and 5 distinction 5 node trees.

Doesn't this kinda look like a doubly linked list? Yes, but there are requirements about where the pointers point. Doubly-linked always point back to each other. Trees can't point back (sometimes you have a pointer to the parent just to be helpful, but this is not technically a part of the structure of the tree).

You can express the original tree with a binary tree by making all left pointers two first child, all right pointers to siblings. We then call the two pointers the oldest child pointer and younger sibling pointer.

```cpp
struct Node
{
    string name;
    Node* oldestChild;
    Node* nextYoungerSibling;
};
```

A **binary search tree (BST)** is empty, or a node with a left binary search tree and a right binary search tree such that the value at every node in the left subtree is <= the value at this node, and the value at every node in the right subtree is >= the value at this node

We can search through this tree really quickly (given the tree is relatively balanced). You can guess the value from $N$ items in $\log_2 N$ guesses. You can also do binary search on an array or vector, but binary trees are faster for insertions and deletion. You can insert and delete here in $\log N$ time, whereas this operation is linear time in indexable data types.

Insertion is a tree algorithm, but you only follow one path so you don't really need recursion.

Deleting is easy for a leaf node. Also pretty straightforward for a node with one child. The whole subtree is less than or greater than the deleted node's parent.

Two children that aren't empty is tougher. Promote one child to replace the deleted parent. You go left one then as far right as you can go, or one step right and far to the left as you can go. Both are good candidates for the promotion.

You keep good performance if the tree stays balanced as you insert and delete.

```cpp
// prints in alphabetical order
void printTree(const Node* p)
{
    if (p != nullptr)
    {
        printTree(p->left);
        cout << p-> name << endl;
        printTree(p->right);
    }
}
```

This algorithm is neither preorder or postorder. The concept of 'between' makes sense for binary trees (exclusively). New term for this is **inorder traversal**.

How to fill the tree? We know the root must have been the first item inserted. It's children must have been the next. You don't know the exact order of insertion by looking at the final tree.

You could get some really bad orderings. If you inserted a sorted list into a BST, you get awful search and other operation efficiency. On average, you get something 6x as long as a perfectly balanced tree.

One solution (non-naive insertion) is an **AVL tree**. At every node, the height of the left and right subtree can differ by no more than one. You keep track of the heights of subtrees, and if an insertion would create an imbalance, you must do a reordering. It's still $\log N$ for insertion and deletion but with a higher constant of proportionality for rebalancing.

Another solution **2-3 Tree**. Nodes with 3 children have 2 values, nodes with 2 children have 1 value. Nodes to the right of a 2-value node are less than both values? (there are complicated rules for this). Every leaf node is at the same height. There is more tricky insertion and deletion but we won't get into these details. Still $\log N$ (even at the worst case) but lower constant of proportionality. Also sometimes $\log_3 N$.

This led into **2-3-4 trees** and even higher numbers (because they all reduce height). In a **2-3-4 tree**, you can represent direct analogies for 1, 2, 3- value nodes in binary tree form. In order to pull this off you need to store a little bit more binary information for each node about what exact BST analogue it has. Instead of true/false they choose red/black. Thus the BST analogue of a 2-3-4 tree is called a **red-black tree**. There's some complicated stuff to transform between the structures. The tree will generally be balanced enough to stay faced. Turns out red-black is faster than 2-3-4 and AVL so most standard libraries use this. 

CPP library types that do quick lookups:

```cpp
#include <set>

set<int> s;
s.insert(10);
s.insert(30);
s.insert(10); //doesn't insert

if (s.find(20) == s.end())
{
    cout << "20 is not in the set";
}
s.insert(5);
for(set<int>::iterator p = s.begin(); p != s.end(); p++)
{
    cout << *p << endl;
} //will write 5 10 30

s.erase(30);
```

A set will visit items in increasing order. The type of item in the set must have a less than operator (and it must meet the logical mathematical less-than results). Note that you don't need `==` to use a set. It tells if an item is a duplicate if `x <= y` and `y <= x`. So be careful with how you define the less than operator.

```cpp
#include <set>
//also same header

multiset<int> s;
//s allows duplicates, still maintains order
```

# Week 8 Wednesday: Hash Tables

What if instead of a 100,000 element array of pointers to student records, we used 10,000 **buckets** with linked lists of elements in them. If we add two elements to the same bucket, this is called a **collision**. Note: just finding a non-empty bucket does not mean that you've found the specific element you're looking for. Also, no point in sorting the linked lists because you want them to be short anyways in a hash table. 

Generally, a data structure that can look up efficiently by one index cannot look up quickly by a second index. Also, integers were particularly convenient to index by because you can index and array and get easy bucket numbers.

In our student ID example, we need elements to be uniformly distributed but these integers aren't, so buckets will be bad (remember that you don't want that many elements per bucket). This is quantified by the **load factor**, which is the number of items divided by the number of buckets.
 $$\text{load factor} = \frac{\text{number of items}}{\text{number of buckets}}$$

 What if we want to make a hashtable with dates? We can just find some integer that maps our dates to 1-10,000. Find an int, then scale down to the number of buckets. This is called a **hash function**.

 Here's a hash function for people's names: add all ASCII character values. But the issue here is that the distribution will not be uniform. One issue is that the hash function only gives values between 0 and 3660.

 To get bigger numbers, we could multiply (and use unsigned integer so that there is well-defined behavior when you roll over). But multiplying is bad because even one even character code will mean you only use one half of the buckets. (Problems with common factors with the scaling number). One solution is to eliminate common factors. Rather than using 10,000, what if we used a large prime number? That keeps it a bit more uniform. Now we just use a strong hash function that returns nicely distributed integers.

 A hash function for strings: Variant of FNV-1 (Fowler-Noll-Vo)

 ```cpp
 unsigned int h = 2166136261u; //too big for int constant, so you add u
 for(char c : theString)
 {
     h += c;
     h += 167777619;
 }
 return h;
 ```

 Here's how you use it:

 ```cpp
 #include <functional>
 using namespace std;

 string s = "hello";
 unsigned int x = std::hash<string>()(s); // overloaded hashObj.operator()
 ```