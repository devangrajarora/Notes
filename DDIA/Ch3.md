# Storage and Retrieval

## Storage engine
* Store the provided data, and return it back when asked
* Implementation can differ based on
  * What are we optimizing for - Transactional workloads, Analytics etc.
  * Data structures - Log structured (LSM trees), Page oriented (B-Trees) etc.

Log: Append only data file

## Index
* Auxiliary data structure built on primary data which stores additional metadata to enable locating primary data faster
* Maintaining index incurs an overhead, especially on writes as the index needs to be updated
* Well chosen index can speed up reads, all indexes slow down writes **[trade-off]**
  * Application owner needs to choose index based on common query patterns
