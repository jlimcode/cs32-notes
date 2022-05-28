# Week 7 Monday Lecture

## Applications of Algorithm Analysis: Sorting

One way to sort: take smallest value and put it at first, then look for the next smallest and put it in its place. (Also works for largest value)

To calculate the efficiency, roughly count the number of times that elements are compared. This comes out to $O(n^2)$. You can also look at worst case, best case, average case. For **selection sort**, every possibility requires the same amount of work.

Another algorithm sorts the elements adjacent against each other as it goes through. Each time you go through swapping, you can stop looking at elements that are in their final correct position (max at end etc). Once no swaps occur, everything is in the right place. This is called **bubble sort**. This has best case $O(n)$ if the array is already sorted. In the worst case, we have to do every step so that's $O(n^2)$. The average case is still $O(n^2)$ but with a smaller constant of proportionality.

Think of being dealt a hand in poker, and each card you get you insert into your hand in the right place. This is called **insertion sort**. On average you look at half the items that come before the item in question in the array. This is also $O(n^2)$ but has a lower constant of proportionality than bubble sort on average. Best case is $O(n)$. Say you have an array where no item is more than 3 items way from where it should be (or any bounding constant). Then you know you don't have to look that far to insert each new element. People had an idea to improve the speed of insertion sort by sorting every $n$ elements to get everything into the general ballpark and then doing a final insertion sort which would be linear time. This method is called **shell sort** (named after the guy who created it). This algorithm is $\approx O(n^{1.5})$.

We've already encountered the **merge sort**. The time complexity for this algorithm (when the list is split evenly) is expressed as a recurrence relation 
$$T(n) = 2T\left( \frac{N}{2} \right) + O(N)$$
Then we can solve this recurrence relation to get the equation
$$T(N) = O(N\log_2 N)$$
This will grow faster than linear, but much slower than $n^2$. There is a big difference for larger amounts of data. 

We'll look at another algorithm with $N\log N$ time. This one is called **quicksort**. We implemented this before with our `separate()` function. You keep splitting into two piles where one pile contains all elements greater than every element in the other pile. This works best when you split the array evenly. But for general cases, you don't know where the midpoint is. So picking an array element randomly works pretty well. What if you calculate the median item first, to guarantee efficiency. This doesn't work because it takes too long to find the median (same time as if you had a bad split). You could sample and find the median of the sample (which gets you pretty close). Sample 3 items is the general best.

# Week 7 Wednesday: Sorting, Trees

Note: merge sort takes that same amount of time best case, worst case, etc. 

## Improving Quicksort

More quicksort: best case is linear. Worst cases are pretty common and include the perfect sorted order or reverse order. These worst cases are $O(N^2)$. quicksort has better constants of proportionality in the general case, so we really want to mitigate the worst cases.

If hackers know what sorting algorithm your server is using, they can construct sequences that are the exact worst case and then DDOS you with those. You can avoid this by picking random pivots every once in a while. But sometimes they can reverse engineer your pseudorandom generator. But you can try to use true random pivot picks (from radiation).

What if you pick a base case of 9 or fewer items? Then when you're done, your pivots are in the right spot, and each of your items is in the right region. Insertion sort is linear time if there's a constant number of steps that bounds the distance from an item to its final position. Research at one point found that 9 is pretty good cutoff point. Still doesn't address the worst case situations but improves quicksort.

We could have quicksort monitor itself, so if the recursion ever gets too deep, we swap over to merge sort. Too deep is around $2\log N$. This improvement is called **introsort** because it's introspective. This algorithm has worst case $O(N\log N)$. Most programming libraries will do introsort in their main sorting algorithm.

## Trees

Some CS concepts are best represented with trees. One thing is a class hierarchy. Another thing is HTML documents. Trees are made of **nodes and edges**. A **path** in the tree is a sequence of edges that connect two nodes. The **root** is the node with no parents (typically a pointer to the top node). **Child nodes** have parents. A true tree must have a unique path from each node to every other node. Trees can't have multiple parents nor loops. **Sibling nodes** have the same parents. A node with no children is called a **leaf node**. Other nodes are called **interior nodes**. **Depth** of a node is how many edges away from the root it is. The **height** of a tree is the depth of the deepest node.

```cpp
struct Node
{
    string name;
    vector<Node*> children;
};

Node* root;

int countNodes(const Node* p)
{
    if (p == nullptr)
        return 0;
    int total = 1;
    for (int k = 0; k < p->children.size(); k++)
    {
        total += countNodes(p->children[k])
    }
    return total;
}
```

You can approach your `countNodes` function in multiple ways:
1. Go all the way down until the node has no more children. Then you come back up and move to the next child. Then you use a stack. A stack seems to suggest recursion. Note the two base cases: empty tree or leaf node. Note that if we're not passed a tree, we'll either get the wrong answer or infinite recursion.
2. I really expected a breadth-first solution but I guess that comes later lol

The takeaway is that trees are very well suited to recursion.

What about printing all the nodes of a tree in the proper order with indentation to indicate depth?

```
Elizabeth
    Charles
        William
            George
            Charlotte
            Louis
        Harry
            Archie
    Anne
        Peter
            Savannah

just as an example...
```

To correctly determine the amount of indentation, we need to know something about our depth. We can modify the parameters to pass the depth. You can construct the string as `string(2* depth, ' ')`. You can overload with the additional arguments and simplify the interface. 

```cpp
void printTree(const Node* p, int depth)
{
    if (p != nullptr) //you can move this call out
    {
        cout << string(2*depth, ' ') << p->name << endl;
        for (int k = 0; k < p->children.size(); k++)
        {
            printTree(p->children[k], depth + 1);
        }
    }
}

void printTree(const Node* p)
{
    printTree(p, 0);
}
```

Slightly better version if you only intend for one version of the function to be called:

```cpp
void printTree(const Node* p, int depth)
{
    cout << string(2*depth, ' ') << p->name << endl;
    for (int k = 0; k < p->children.size(); k++)
    {
        printTree(p->children[k], depth + 1);
    }
}

void printTree(const Node* p)
{
    if (p != nullptr)
    {
        printTree(p, 0);
    }
}
```