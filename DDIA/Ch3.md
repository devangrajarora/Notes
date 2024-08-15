# Storage and Retrieval

## Storage engine
* Store the provided data, and return it back when asked
* Implementation can differ based on
  * What are we optimizing for - Transactional workloads, Analytics etc.
  * Data structures - Log structured (LSM trees), Page oriented (B-Trees) etc.

Log: Append only data file

## Indexes
* Auxiliary data structure built on primary data which stores additional metadata to enable locating primary data faster
* Maintaining index incurs an overhead, especially on writes as the index needs to be updated
* Well chosen index can speed up reads, all indexes slow down writes **[trade-off]**
  * Application owner needs to choose index based on common query patterns

## Bitcask: Log storage with Hash Index
* On-disk log storage with in-memory hash table
* Key value store. High performance reads & writes, keys should fit in RAM.
  * Insertion: append only log (on disk)
  * Updates: New insertion
  * Retrieval: Using a hash index - store byte offset of key value pair in a hashmap (in memory)
  * Number of unique keys should be small enough to be held in memory
* Values can be large and are stored on disk. Caching can help in retrieving values faster
* Good for large number of updates as compared to new entries

### Implementation
#### Compaction
* Problem: We might run out of disk space if we never clean up old entries in an append only system
* Compaction can be performed once a data file reaches a certain size
* Process of writing the most recent updates for the keys to a new file segment. Since the number of unique keys is small, we can merge many data segments into a smaller segment
  * Compaction writes data to a new segment. At the end old segments can be discarded
* This can be done on a background thread while the system continues to serve read requests using old segments and write requests on the latest segment file
* Once the compaction is done, the merged segment file can be used to serve read requests instead of the older ones
  * Older segment files can be deleted at this point
* Each segment has its own hashmap with byte offsets for the given keys
* Lookups check the most recent segment file first and so on
#### File format
* Binary file format
* Encode length of string in bytes first and then string
#### Deleting a record
* Append a special deletion record to file (aka tombstone) for the key. Hashmap value for the key can be cleared
* While merging entries before the tombstone are ignored
#### Crash Recovery
* In-memory hashmap will be lost during crash. We can rebuild the hashmap from the data file which is on disk
* Bitcask stores a snapshot of the hashmap of each data file which can be loaded into memory on server startup
#### Partially written records
* DB might crash in the middle of writing a record
* Bitcask files include checksums which allows corrupted parts of files to be ignored
#### Concurrency Control
* Reading: Multiple threads can read data concurrently since file is append only
* Writing: Common practice is to have only one writer thread
### Analysis of an append only system
Pros
* Concurrency and crash recovery are simpler
* Appending and segment merging are sequential write operations which are more efficient that random writes on both HDD and SSD
* Merging old data files prevents excess disk space usage and risk of files getting fragmented over time

Cons
* Hash table must fit in memory
* Range scans are not effective. Each key needs to be search individually

## SSTables and LSM-Trees
### SSTables
* Sorted String Table (SSTable): KV pairs are sorted by key
* Merging segments
  * Simple and efficient, similar to mergesort
  * Number of unique keys need not fit in memory
* Sparse index
  * No longer need to store all keys in memory (like the hash index in Bitcask)
  * Store some keys (one for every few kB of the segment, pretty fast scan) with their byte offset
* Compression
  * Since read requires a (small) range scan, it is possible to group the records in a block and compress them before writing to disk
  * This saves disk space and I/O bandwidth
 
### Constructing and maintaining SSTables
* In-memory balanced tree to maintain sorted order of incoming writes
  * aka Memtable
  * Implementations: Red-Black trees, AVL trees
* Incoming read and write requests are served by the memtable
* Once the memtable gets larger than a certain threshold (typically greater than a few MB), write it to disk as an SSTable
  * While this process in going on, new writes go to new memtable instance
* For incoming reads, first search memtable, then the SSTable file starting from most recent
* From time to time run compaction algorithm to merge SSTables and discard old ones
* Searching in the SSTable
  * Sparse index is written to the same file
  * Load index keys in memory and search for target key (binary search)
  * Read file starting from the derived byte offset

### LSM Tree
* The above described storage structure is referred to as Log-Structured Merge Tree
* Core idea: Keeps a bunch of sorted files (eg - SSTable files) and performs merging and compaction on them in the background
* Range queries are efficient since keys are sorted
* Merge can support high write throughput as data is to be written sequentially

## B-Trees
* Most widely used indexing structure
* Keeps keys in sorted order - efficient key value lookups and range queries
* Can be implemented to store data on disk
* Stores keys and records in fixed size pages (~4kB each)
  * Pages are read and written together
  * Page is composed of multiple filesystem blocks
  * Pages can be identified by address or location on disk, which allows one page to point to another
* Types of pages
  * Root page: Start of B-Tree
  * Inner page: Contains range boundaries (separator keys) and references to other pages between to keys. The reference points to another page which would contain keys between k<sub>1</sub> and k<sub>2</sub>
  * Leaf page: Contains keys and values (values can be inline or we would have a reference to a page which contains the value)
* Branching factor
  * The number of children of a page
  * Depends upon the space occupied by page references and separator keys/range boundaries
* Updating a key
  * Locate key in leaf node and update value
  * Write the page back to disk with updated value
  * Address/offset of the pages remains the same
* Adding a new key
  * Locate leaf page to add key in
  * If space available, add the key and value
  * If not, split the page into two half-full pages, and add a pointer to the newly created page in the parent. Update the separator key in the parent
  * If parent is full this propagates upwards
* Depth of B-Tree is O(logN)

### In place updates and reliability
* B-Tree allows updating content of a page and rewriting the page
  * This is contrary to LSM trees which are append only
  * Page write is a hardware operation
* Updates don’t change pointers to the page
* Multi-page updates
  * Some updates require multiple pages to be rewritten. Example: Splitting a page
    * Creating new page and moving half data to it
    * Removing half data from current page
    * Adding new pointer to parent
  * If DB crashes during this operation, index could end up in a corrupted state
  * To prevent this DB has a write ahead log
* Write Ahead Log
  * Every operation to a DB is written to a log before it is executed
  * Log is used to restore changes if DB crashes mid operation
* Concurrency is important while writing to a page. This is typically handled using latches (lightweight locks)

### Optimizations
* Abbreviate keys(esp inner nodes): Decrease key size, more keys in page, higher branching factor
  * This is used in B+ Trees
* Increase locality: Keep pages together on disk, better for large range queries by reducing disk seeks
* Increase pointers: Leaf nodes can have front and back pointers to siblings

## Comparing B-Trees and LSM Trees
* General rule of thumb: B-Trees faster for reads, LSM Trees faster for writes
  * LSM Trees have to check multiple data structures and SSTables at different stages of compaction
* In reality this depends on specific workloads

### Advantages of LSM Trees
* Write amplification
  * Process of one write to database leading to multiple writes to disk over the DB’s lifetime
  * B-Trees: Record is written to page and WAL. Rows are also written again during splitting and merging
  * LSM Trees: Rows are rewritten to disk during merging and compaction of SSTables
  * Especially harmful in SSDs where overriding blocks leads to block getting worn out over time
  * Can be bottleneck in write heavy applications
  * Higher writes lead to higher consumption of disk I/O bandwidth
* LSM Trees typically have lower write amplification
* LSM Trees can support higher write throughput
  * Lower write amplification
  * Sequential writes (faster in both HDD and SSD)
* Better disk space utilization
  * In LSM Trees, keys can be compressed and take lesser space on disk
  * B-Trees typically have fragmented disk space within pages (empty slots, half filled pages after split)
  * Lower storage overhead

### Disadvantages of LSM Trees
* Large amount of write bandwidth of disk can be consumed during compaction which can interfere with incoming queries to the DB
  * This is not a big problem for most queries but spikes up P99 latency
  * Thus compaction can interfere with read/write performance
  * If writes slow down → more segments exist since compaction slows down → more segments need to be checked → reads slow down
* Same key might exist across multiple segments with different values
  * On the contrary, B-Tree only stores one copy of a key

## Other indexing structures
* KV indexes like the ones discussed above are used to build primary key index (keys are unique)
* Secondary key index: Index on other fields in the record
  * Important for joins, ex: join on user_id would be faster if a secondary key exists on user_id
  * Keys may not be unique. 2 solutions
    * Create postings list for each key (list of values)
    * Make keys unique by adding row number as prefix
  * Can be built using both B-Trees and LSM Trees

### Storing values in the index
* Index can store values inline (clustered index) or a reference to the value (non-clustered index) which is written somewhere else
* Heap files
  * Values can be written to heap files and pointed to by the index
  * Values/rows are stored in random order
  * Heap file might be append only and has to keep track of deleted rows to reclaim the disk space
  * Multiple indexes (primary and secondary) can point to the same location in the heap file. This avoids duplication of rows
  * Updating value for the same key
    * If the new value is smaller than the original, we can update in the same location. Index doesn’t need to be updated
    * If new value is greater, we need to write to new memory location and update the index
* Clustered index
  * Saves extra disk seek from index file to heap file during reads
  * More disk memory consumption, additional storage overhead
  * InnoDB storage engine in MySQL
    * Primary key index is clustered index
    * Secondary indexes refer to primary key
* Covering index
  * Middle ground between clustered and non-clustered index
  * Stores some columns with the key
  * Allows queries on some fields to be faster and directly handled by primary index

### Multi-column indexes
* If we need to query rows based on a single column (key), the above described indexes work, but not for queries which span multiple columns
* Concatenated index
  * Concatenate columns into one key
  * Can be used to query over any prefix of list of keys, but not for individual or random group of keys
* Multidimensional index
  * Values for multiple columns can be converted to a single value using a _space filling curve_ which can then be used to create the index

## In-memory databases
* Issues with storing data in memory
  * RAM is expensive
  * Not durable, data lost in case of crash
* With technological advancements, cost of RAM going down
* Combating durability concerns
  * Battery powered RAM
  * Maintaining log on disk storing DB operations to load data back to memory
  * Storing snapshots of DB to disk
  * Replication to other machines
* More complex data structures can be used in memory, and we avoid overhead of converting them to a disk based representation
  * This also allows allowing use of more complex data models
  * Redis provides priority queues, sets etc
