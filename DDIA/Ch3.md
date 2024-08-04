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

