# note of chapter 17



## 17.1 Assumptions

1. we are concerned with external fragmentation
   * of course we can have internal fragmentation(if an allocator hands out chunks of memory bigger than that requested), but for the sake of simplicity, mostly focus on external fragmentation
2. assume a basic interface such as that provided by *malloc()* and *free()*
3. once memory is handed out to a client, it cannot be relocated to another location in memory(no **compaction** is possible)
4. the allocator manages a contiguous region of bytes 





## 17.2 Low-level Mechanisms

#### Splitting and Coalescing

1. splitting: request for something smaller than the size of any one in the free list
   * it will find a free chunk of memory that can satisfy the request and split it into two. 
   * the first chunk it will return to the caller; the second chunk will remain on the free list
2. coalescing: when returning a free chunk in memory, look carefully at the addresses of the chunk you are returning as well as the nearby chunks of free space; if the newly-freed space sits right next to one(or two) existing free chunks,merge them into a single larger free chunk

#### Tracking The Size Of Allocated Regions

*free()* don't need a size parameter



1. **header block**: usually just before the handed-out chunk of memory, containing:
   1. size
   2. additional pointers to speed up deallocation
   3. a magic number to provide additional integrity checking
   4. other information

#### Embedding A Free List



#### Growing The Heap



## 17.3 Basic Strategies

#### Best Fit

first, search through the free list and find chunks of free memory that are as big or bigger than the requested size. Then, return the one that is the smallest in that group of candidates



#### Worst Fit

find the largest chunk and return the requested amount; keep the remaining (large) chunk on the free list



#### First Fit

finds the first block that is big enough and returns the requested amount to the user.



#### Next Fit

 keeps an extra pointer to the location within the list where one was looking last. The idea is to spread the searches for free space throughout the list more uniformly, thus avoiding splintering of the beginning of the list





## 17.4 Other Approaches

#### Segregated Lists

1. basic idea: if a particular application has one (or a few) popular-sized request that it makes, keep a separate list just to manage objects of that size; all other requests are forwarded to a more general memory allocator





#### Buddy Allocation

