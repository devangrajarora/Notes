# B-Tree Basics

## Mutable Storage structures
* In-place update mechanism
* Data records are updated directly at their location in the target file during insertion, updation and  deletion operations
## Binary Search Tree
* Sorted in-memory data structure
* Each node has a key and a value. A node is identified by its key and keys should be unique.
* Each node splits the search space into 2 parts:
  * Left subtree: All keys are smaller than the root key
  * Right subtree: All keys are larger than the root key
* Key lookups: O(log<sub>2</sub>N) for balanced BST, O(N) for unbalanced BST
  * Balanced BST: Height difference between 2 subtrees <= 1
* Tree must be balanced after each operation to maintain lookup efficiency
  * Balancing: Keeping height and number of nodes in left and right subtree in bounds
  * Balancing method: If an insertion leaves a branch having 2 consecutive nodes with only one child, rotate the branch along the middle node

### BST as a disk based data structure
#### Issues
* Worst case linear complexity
* Balancing requires pointer updates, node relocation
* Low fanout makes balancing frequent
  * Fanout: Maximum number of allowed children per node
* Increased maintenance costs make BST impractical as on-disk data structures
* Locality: Since insertion order is random, there is no guarantee that child pointer will be close to its parent in memory. Child nodes could span multiple pages
* Low fanout → Increased height of BST → more disk seeks per search due to lack of locality

#### Improvements needed
* Higher fanout → Increased locality between neighbouring nodes
* Lower height → Reduce number of disk seeks during traversal

Overall BST is not suited as a disk based data structure and works better as an in-memory data structure.

## Disk based data structures
Some data structures are better suited to be used on disk and some in memory

| Disk  | In-Memory |
| ------------- | ------------- |
| Store large amounts of data  | Can only store little data  |
| Updates are more expensive  | Quick updates and easier pointer access  |

### HDD
* Hard disk drives/Spinning disks
* Seeks require disk rotation and mechanical head movement to place the read/write head at the position from where a contiguous memory block will be read/written
  * Sector - smallest unit of read/write (512 bytes - 4Kb)
* Head positioning is expensive, reading/writing to sector is relatively cheap

### SSD
* Solid state drive
* No disk rotation or head positioning is required
* SSD is made from memory cells. Memory cells hold a one or more bits
  * Cells are connected into strings
  * Strings are connected into arrays
  * Arrays are combined into pages
  * Pages are combined into blocks
  * Blocks are combined into planes
  * Planes are placed on a Die
* Smallest unit that can be written or read is a Page

<ins>Challenges for making a Disk based data structure</ins>
* Disk seek cost
* Block level abstraction
  * Smallest unit of data extraction is a block. If our data structure points to a certain memory location, we will have to read the whole block. The data structure should align with this architectural limitation.
* Optimize for fewer disk accesses
  * Improving locality
  * Reduce out-of-page pointers
  * Optimize internal memory representation

## B-Trees
* Nodes are represented as rectangles with pointer blocks
* More data can be stored in the same node → Increased fanout, decreased height
* B-Trees are sorted
  * Each node contains multiple keys which are sorted. We can use binary search to search within nodes
  * Can be used to perform both point (single item) and range queries
* B-Trees are a page organization technique, node and page are used interchangeably

* Hierarchy
  * Each B-Tree node contains multiple keys
  * Each node contains N keys and N+1 child node pointers
  * Root node, internal nodes, leaf nodes

* Occupancy: Relationship between node capacity and number of keys it actually holds
* Fanout
  * B-Trees are characterized by their fanout
  * Higher fanout helps amortize the cost of disk seeks and tree balancing

* Tree balancing (split, merge) is triggered when node is full or nearly empty
* B+ Trees
  * Actual implementation of B-Trees
  * Data is only stored in leaf nodes
  * Internal nodes just store separator keys
  * Operations (insertion, deletion, updates) only affect leaf nodes and only propagate upwards during split and merges
  
* Separator Keys
  * Keys stored in B-Trees are called index keys, separator keys or divider cells
  * Split the tree into subtrees holding key ranges

* Keys are stored in sorted order
  * First pointer in a node points to a subtree with keys strictly smaller than the first key in the node
  * Last pointer in a node points to a subtree with keys greater or equal to the last key in a node
  * K<sub>i-1</sub> ≤ K<sub>s</sub> < K<sub>i+1</sub> where K<sub>s</sub> is a key that belongs to the subtree

* Some B-Tree variants have sibling pointers between leaf nodes (single or double sided)
* B-Trees are built bottom up
  * As leaf nodes get filled up, we need to add more internal nodes
  * This is contrary to BSTs

### B-Tree Lookup Complexity
* 2 factors
  * Number of block transfers
  * Number of comparisons

M → number of items in tree
N → number of keys per node
K → number of times more nodes on each level

* Number of child pointers to be followed from root to leaf is height of tree
* At most log<sub>K</sub>M pages are addressed to find the search key
* Lookup complexity → log<sub>2</sub>M ~ logM

### Insertion
* For insertion - locate target leaf and find insertion point. If node has empty elements, insert the new element
* Split needs to be performed if
  * Node can hold N keys and adding a new key takes the count over N
  * Index separator can hold N+1 pointers and adding new key takes number of pointers over N+1
* Split process:
  * Allocate new node
  * Move half the elements to new node
  * Insert new element in one of the nodes
    * If inserted key is less than the key at splitting point, insert into split node
    * Else insert into new node
  * At the parent of the split node, add a separator key and pointer to the new node
    * If the parent is at full capacity, it needs to be split similarly
    * This split could propagate up to the root. In that case a new root is added and height of tree increases by 1

### Deletion
* For deletion - locate target leaf and target key. Delete key and value
* Merges
  * If two neighbouring sibling nodes have too low occupancy, their contents are merged into one
  * Merge conditions
    * For leaf nodes: if a node can hold up to N key-value pairs, and combined number of key-value pairs in two adjacent nodes in less than or equal to N
    * For non-leaf nodes: if a node can hold N+1 pointers, and the combined number of pointers in two adjacent nodes is less than or equal to N+1
* Merge process:
  * Copy all elements from right node to left one
  * Remove right node pointer from parent
  * Remove right node
