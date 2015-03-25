# Building-Blocks - Generic Container

Most data structures are largely independent of the data stored in
them. A list can be used to store a number of integers, doubles,
threads or even other lists. While the details in each case might
differ the overall structure and code is always the same. Lists,
Trees, Sets, Maps, Hashtabels are all generic containers.

Containers can be implemented in various ways. Each has its own
benefits and drawbacks. Here are a few (using a single linked list as
example):

Detached
--------

In a detached container the data structures control structure (Item)
is detached from them payload (Data). The Items only contain a pointer
to the data itself but no actual data.

  +-Item-+    +-Item-+    +-Item-+    +-Item-+    +-Item-+
  | next ---->| next ---->| next ---->| next ---->| next ---->
  | data --+  | data --+  | data --+  | data --+  | data --+
  +------+ |  +------+ |  +------+ |  +------+ |  +------+ |
           v           v           v           v           v
       +-Data-+    +-Data-+    +-Data-+    +-Data-+    +-Data-+
       | data |    | data |    | data |    | data |    | data |
       +------+    +------+    +------+    +------+    +------+

Items are always the same size no matter how large the data is. The
data an item points to can also be easily swapped for another. All it
needs it setting the pointer, which can be done atomically. One
problem is that this link is unidirectional. Given a Data structure
there is no way to get the Item that points to it. Therefore you
sometimes see links from the Data back to its Item:

  +-Item-+    +-Item-+    +-Item-+    +-Item-+    +-Item-+
  | next ---->| next ---->| next ---->| next ---->| next ---->
  | data --+  | data --+  | data --+  | data --+  | data --+
  +------+ |  +------+ |  +------+ |  +------+ |  +------+ |
     ^     v     ^     v     ^     v     ^     v     ^     v
     | +-Data-+  | +-Data-+  | +-Data-+  | +-Data-+  | +-Data-+
     +-- item |  +-- item |  +-- item |  +-- item |  +-- item |
       | data |    | data |    | data |    | data |    | data |
       +------+    +------+    +------+    +------+    +------+

Since now there are two links swapping items is no longer atomic and
in a multi-threaded environment care must be taken to avoid race
conditions.

The detached control structure has a drawback though. It needs to be
separately allocated, causing two allocations per data. Sometimes this
is a plus though. The data can have special requirements on alignment
or be allocated from somewhere else that makes using a joined or
internal container impractical.

Joined
------

A drawback of the detached container are the separate allocations. But
why should one do 2 allocations just because there are 2 structures?
One can just allocate a larger chunk and put both of them in there:

    +--------+      +--------+      +--------+      +--------+      +--------+
    |+-Item-+|      |+-Item-+|      |+-Item-+|      |+-Item-+|      |+-Item-+|
    || next -------->| next -------->| next -------->| next -------->| next --->
  +->| data ---+  +->| data ---+  +->| data ---+  +->| data ---+  +->| data ---+
  | |+------+| |  | |+------+| |  | |+------+| |  | |+------+| |  | |+------+| |
  | |+-Data-+<-+  | |+-Data-+<-+  | |+-Data-+<-+  | |+-Data-+<-+  | |+-Data-+<-+
  +--- item ||    +--- item ||    +--- item ||    +--- item ||    +--- item ||
    || data ||      || data ||      || data ||      || data ||      || data ||
    |+------+|      |+------+|      |+------+|      |+------+|      |+------+|
    +--------+      +--------+      +--------+      +--------+      +--------+

This half's the number of allocations needed while still meaning that
moving Data between Items is just changing 2 pointers. There will
always be exactly as many Items and Data (or an integer multiple if
one allocates space for n Items per Data). Freeing an Item or Data
becomes a little more complex since they can only be freed in
pairs. If the Data has been shuffled around between the Items then
the Item in the same block as the Data might not be linked. But usually
the Item is still small and exchanging the Item in the same block as
the Data for the Item the Data points to is easy. And once exchange
the block can be freed as one.

