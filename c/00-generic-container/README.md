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

In a detached container the data structures control structure (Node)
is detached from them payload (Data). The Nodes only contain a pointer
to the data itself but no actual data.

![Deatched container](detached1.png)

Nodes are always the same size no matter how large the data is. The
data an node points to can also be easily swapped for another. All it
needs it setting the pointer, which can be done atomically. One
problem is that this link is unidirectional. Given a Data structure
there is no way to get the Node that points to it. Therefore you
sometimes see links from the Data back to its Node:

![Deatched container with backlinks](detached2.png)

Since now there are two links swapping nodes is no longer atomic and
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

![Joined container with backlinks](joined.png)

This half's the number of allocations needed while still meaning that
moving Data between Nodes is just changing 2 pointers. There will
always be exactly as many Nodes and Data (or an integer multiple if
one allocates space for n Nodes per Data). Freeing an Node or Data
becomes a little more complex since they can only be freed in
pairs. If the Data has been shuffled around between the Nodes then
the Node in the same block as the Data might not be linked. But usually
the Node is still small and exchanging the Node in the same block as
the Data for the Node the Data points to is easy. And once exchange
the block can be freed as one.

