### Indexing

#### # Terms

- Asymptotic - method for describing limiting behavior. f(n) = n^3 + 3n. As n approaches infinity, 3n becomes insignificant compared to n^3.
- https://github.com/phishman3579/java-algorithms-implementation

#### # Trie

https://medium.com/basecs/trying-to-understand-tries-3ec6bede0014

https://github.com/eugenp/tutorials/tree/master/data-structures/src

Also means Re *trie* val. It is a tree data structure wherein the nodes of the tree store the entire alphabet, and strings / words can be retrieved by transversing down the path of the tree. 

![img](https://miro.medium.com/max/3264/1*sZOrNXzlQICVv5ePpav1-g.jpeg)

#### # B-Tree

- Internal nodes (not root, not leaf) can have a variable number of child nodes within a pre-defined ranged. The lower and upper bound of the range are fixed for particular implementations.

- b-tree of order of `m` simply means a max of `m` children and max of `m-1` keys in a node. B-tree  has **6 properties**:
  - All keys in a node must be in ascending order.
  - All leaf nodes must be at the same level.
  - The standard rule is that a node must atleast have a minimum of `[m/2]-1` keys and a max of `m-1` keys. There are exceptions for root and leaf nodes.
  - All non-leaf nodes except root must have atleast `m/2` children.
  - A non-leaf node with `m-1` keys must have `m` children. 
  - If a root is a non-leaf, it must have atleast 2 children.
  
  ##### Example
  
  B-tree of order of 4:
  

| Node | Leaf (children = 0) | Non-leaf (children > 0) |
| ---- | ---- | -------- |
| Root | Keys: 1 | Keys: 1-3<br />Children: 2-4 |
| Internal node | Not applicable | Keys: 1-3<br />Children: 2-4 |
| Leaf (non-root) | Keys: 1-3 | Not applicable |

#### # Invert Index

Store Field DocValues

N-Dimensional

Merge-Sort



#### # Apache Lucene

Document -> analysis -> Token or Term -> indexing -> Inverted Index

- Token is a atomic set of information (e.g. a word etc)
- How a token or token is stored in index is called Term Vector



#### # Index Data Structures

~~**Trie**~~
- https://medium.com/basecs/trying-to-understand-tries-3ec6bede0014
- https://github.com/eugenp/tutorials/tree/master/data-structures/src

**Suffix or Compressed Trie** 
- https://www.hackerearth.com/practice/data-structures/advanced-data-structures/suffix-trees/tutorial/
- Ukkonen's Algorithm
- https://www.cs.cmu.edu/~ckingsf/bioinfo-lectures/suffixtrees.pdf
- https://www.geeksforgeeks.org/ukkonens-suffix-tree-construction-part-1/

Suffix Trees
Suffix Array
Borrows-Wheeler transform
FM Index

Radix

Hash

2-3 Tree

ADT

AA Tree

AVL Tree

B-Tree

Red-Black Tree

Scapegoat Tree

Splay Tree

Treap

Weight-Balanced Tree

K-D Tree

Hash

Heap

Binary Search Tree

Ternary search trees

Skip List

Sorted Linked List

Unsorted Linked List

Queue

Stack

Sorted Array

UnSorted Array

