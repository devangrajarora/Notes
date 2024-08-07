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
