# File Formats

Main (primary) memory
* Volatile, fast access
* RAM, ROM

Disk (secondary) memory
* Non-volatile, slow access
* Can’t be accessed directly by the computer
* HDD, SSD

Virtual memory
* Memory management technique through which secondary memory can be accessed as though its a part of the main memory
* No need to manage offsets manually

On-disk B-Trees
* Page management mechanism
* Algorithms have to create and navigate pages

### Challenges with storing data on disk
* Manage garbage collection and fragmentation of data
* Allocate data in a way to allow quick access
* Consider specifics of the persistent medium
* Serialize and deserialize efficiently
* Come up with binary data formats

## Binary Encoding
* To store data on disk, we need to encode data in a format which allows easy serialization/deserialization
* Efficient page layout is important
* Need to represent keys and data records in binary
* Only byte sequences can be stored on disk/transferred over network
  * Serialization: Represent key, values, and complex types in their binary format (byte sequence) 
  * Deserialization: Convert byte sequences back to keys, values and complex types

### Endianness
* Relative ordering of **bytes**
* Little Endian
  * LSB (byte) comes first, followed by increasing order of significance
  * LSB has lowest address in memory
* Big Endian
  * MSB comes first, followed by decreasing order of significance
  * MSB has lowest address in memory
* Same endianness needs to be used while encoding and decoding data

Consider a 32-bit integer represented by 4 bytes B<sub>1</sub>B<sub>2</sub>B<sub>3</sub>B<sub>4</sub>\
Little endian -> B<sub>4</sub>B<sub>3</sub>B<sub>2</sub>B<sub>1</sub>\
Big endian -> B<sub>1</sub>B<sub>2</sub>B<sub>3</sub>B<sub>4</sub>

### Primitive types
* Occupy fixed space in memory
* byte - 8 bits; short - 2 bytes, int - 4 bytes, long - 8 bytes
* float and double are represented by sign, exponent and fraction
* Use primitive types to create complex types, fixed size arrays, pointers to other addresses

### Strings and Variable-Size data
* Strings and arrays (of fixed size elements) can be encoded using their size in bytes and the byte array
* This is called Pascal String in case of string
* Alternative: null terminated string
```
String
{
 size uint_16
 data byte[size] 
}
```

### Bit-packed data
* Booleans - can be represented using 1 byte, but single bit is enough
  * We can batch booleans together (8 boolean values in 1 byte)
* Enums
  * Often-repeated low-cardinality values
  * Can be represented as integers

## File Structure
* Files are generally split into fixed sized pages
* Each page is a single block or contiguous blocks
* Using pages of same size allows easier read and write access
* Files typically have a fixed-sized header and trailer which contain information which needs to be accessed quickly
* Rest of the file is split into pages
* File format for a schema
  * name string, int age, string address
  * Store fixed size fields first, variable sized later
  * Variable sized fields start with the byte size occupied by them
  * Field positions can be identified through offsets

## Page Structure
* DB stores records in data and index files
* Files are divided into fixed sized pages, each page spans multiple filesystem blocks
* B-Tree node is a page or chain of pages
* Each node contains series of triplets of key, value, pointer

## Slotted Pages
* Organization technique used to store variable sized data, eg - strings, BLOBs (Binary Large Objects) etc
* Reclaiming freed up space is an issue with variable size data
  * Consider replacing a record of N bytes with an updated record of M bytes
  * if M < N, N-M space will be unused unless we find another record which takes exactly that much space
  * if M > N, record can not fit in this space and will need a new space, space occupied by old record is not reclaimed
* Splitting page into X sized segments (essentially smaller minimum unit of storage)
  * Unless M % X == 0, we have a partially filled page
  * X - M%X space is left unoccupied
* Page is divided into slots (or cells)
* Page stores a fixed-sized header and a pointer array which points to cells
  * Header stores important information about the page
* Insertion: Store record in available memory and add a pointer to this memory
* Deletion: Nullify the pointer or remove it. Pointers can be reorganized to reserve the order

| Requirements | How Slotted Page technique addresses it |
| ------------- | ------------- |
Store variable sized data with minimal overhead | The only overhead is maintaining the pointer array
Reclaim freed up space | Page can rewritten to consume freed up memory
Address records in page from outside without regard to the exact location | From outside a cell is accessed by the ID, the actual memory location is only known inside the page

### Cell Layout
* 2 kinds of cells
  * Key cells
    * Found in inner nodes
    * Contains separator keys and pointer to the page which lies between two adjacent keys
  * Key-Value cells
    * Found in leaf nodes
    * Contain key and record
* Pages are homogenous - they have either only key cells or only key-value cells
  * also, cells in a Page would either have only fixed size data or only variable sized data, not both
  * since this in consistent at a page level, we can store this information in the page header (ref Slotted Page structure)
* Key cells
  * key_size (int)
  * child_page_id (int)
  * key_bytes (bytes [])
* Key-Value cells
  * key_size (int)
  * value_size (int)
  * key_bytes (bytes [])
  * value_bytes (bytes [])
* We don’t need to store the offset for each child page, instead we can just store the page ID. A lookup table can map page ID to offset in memory
  * Offset for each cell is determined at page level

### Insertion into Slotted Page
* Incoming records are appended at the right end of the cell, and new pointers are appended to the left
  * Both these collections grow towards the center
* Records are added in insertion order but the pointers can be rearranged to maintain the logical order and allow binary search
* Pointers after the new pointer insertion point is moved rightwards

### Deletion in Slotted Page
* Record is not removed from disk, instead is marked for deletion. The pointer to the cell can be deleted.
* The “freed up memory” is updated in an in-memory availability list which stores a pointer to the memory and the available bytes
* For an incoming record we check if the fragmented memory has a fitting place for the new record using the availability list
  * First fit strategy
    * Insert record in the first segment large enough to hold the record
    * Can lead to large overhead since an insertion might render rest of the free space unusable
  * Best fit strategy
    * Insert record in the smallest segment which can hold this record
    * Need to scan all fragments every time
* If no fragment is available but cumulative free space is enough to hold record, we defragment the page by rewriting it
* If not enough space is available (regardless of defragmentation), we create an overflow page

## Versioning
* File formats need versions since they might evolve over time
* Storage engine could have to support multiple file format versions
* This info can be stored in
  * Separate file
  * File header
  * Prefix of file name

## Checksumming
* Files on disk could get corrupted due to hardware and software failures, bugs etc
* Checksumming, Cyclic Redundancy Checks are used to find data corruption
* Checksums
  * We can compute checksum of a page when it is written along with the data
  * Checksum is stored in page header
  * We recompute checksum while reading a page, in case checksum is different we discard the page
  * Checksum can detect single bit change but can’t reliably detect multiple bit change
  * Weak guarantee
* CRC
  * Can be used to tell when a sequence of bits have changed
  * Uses lookup tables and polynomial division
