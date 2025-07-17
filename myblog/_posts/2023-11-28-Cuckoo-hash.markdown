Cuckoo hashing is a dictionary lookup algorithm introduced by Rasmus Phagh and
Flemming Rodler.
The space usage is similar to that of binary search trees.
This data structure is interesting in that it does
not use perfect hashing, but rather a variant of open addressing where keys can be
moved back in their probe sequences. It is competitive
with the best known dictionaries having an average case (but no nontrivial worst
case) guarantee on lookup time.

The most efficient dictionaries, in theory and in practice, are based on hash-
ing techniques. The main performance parameters are of course lookup time,
update time, and space. The constant factors involved are crucial for many
applications. In particular, lookup time is a critical parameter. It is well known
that, by using a simple universal hash function, the expected number of mem-
ory probes for all dictionary operations can be made arbitrarily close to 1 if
a sufficiently sparse hash table is used. Therefore the challenge is to combine
speed with a reasonable space usage. In particular, we only consider schemes
using O(n) words of space. The space usage of cuckoo hashing is roughly 2n
words (similar to binary search tree). The special feature is that the lookup
causes atmost 2 memory accesses.


A basic cuckoo hash table consists of an array of buckets where each item has two candidate buckets determined by hash functions h1(x) and
h2(x).
The lookup procedure looks at both the buckets to check if EITHER contains
this item. The paper "Cuckoo Filter Practically better than Bloom"
shows the example of
inserting a new item x in to a hash table of 8 buckets, where
x can be placed in either buckets 2 or 6. If either of x’s two
buckets is empty, the algorithm inserts x to that free bucket
and the insertion completes. If neither bucket has space,
as is the case in this example, the item selects one of the
candidate buckets (e.g., bucket 6), kicks out the existing item
(in this case “a”) and re-inserts this victim item to its own
alternate location. In their example, displacing “a” triggers
another relocation that kicks existing item “c” from bucket 4
to bucket 1. This procedure may repeat until a vacant bucket
is found as illustrated in Figure 1(b), or until a maximum
number of displacements is reached (e.g., 500 times in our
implementation). If no vacant bucket is found, this hash table
is considered too full to insert. Although cuckoo hashing may
execute a sequence of displacements, its amortized insertion
time is O(1).

Cuckoo hashing ensures high space occupancy because it
refines earlier item-placement decisions when inserting new
items. Most practical implementations of cuckoo hashing
extend the basic description above by using buckets that hold
multiple items.

Sanchez et al used Cuckoo has to find membership information in a few applications. To support
transactional memory, Sanchez et al. proposed to store the
read/write set of memory addresses of each transaction in
a cuckoo hash table, and to convert this table to Bloom filters when full.


Despite the fact that Cuckoo hash provides better theoretical bounds than
linear probing, in practise it performs worser than it.
This is because since the two hash values are at different memory locations,
they cause 2 cache misses. Linear probing can cause a single cache miss;
consequitive probes lie on the same cache line as in case of collision, the
next location in the hash table is used to store the value.
This paper shows the benchmark analysis of this:
http://jakubiuk.net/stuff/hash_tables_cache_performance.pdf
Our experiments show that theoretical analysis is not sufficient to predict algorithm’s be-
havior in practice. In particular, standard running time analysis does not take into account
cache behavior, which has significant impact on performance.
