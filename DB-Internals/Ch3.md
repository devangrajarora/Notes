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
* Relative ordering of bytes
* Little Endian
  * LSB comes first, followed by increasing order of significance
  * LSB has lowest address in memory
* Big Endian
  * MSB comes first, followed by decreasing order of significance
  * MSB has lowest address in memory
* Same endianness needs to be used while encoding and decoding data

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
