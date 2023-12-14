Bloom filters are an excellent data structure for succinctly representing a
set in order to support membership queries.

A Bloom filter is used to represent a set S = {s1, s2, s3...., sn} of n
elements from a Universe U.  This requires |log U| bits per element for
representation.  To transmit n elements we need n |log U| bits. A bloom filter
consists of an array of m bits. Initially all these bits are 0.
Generall m/n is a constant, determined by design for the application. 
A bloom filter uses k different, independent hash functions {h1, h2, ...hk}
with range {0,..., m-1}. To check if an element x is a part of the set, we
check whether all the hash functions evaluate to 1. If not, clearly the
element x is not a part of the Set. However, if all hashes to evaluate to 1,
we have a probability that the element may not be in the set.

Thus a bloom filter provides false positives, but never false negatives.
For many applications this is acceptable as long as the probability of false
positive is small. 

There are 3 fundamental performance metrics for Bloom filter that can be
traded off:
1) computation time of the k hash functions.
2) size of the array; each element is m bits. One element per element in the
set.
3) Probability of false positive.

Using more hash functions lowers the probability of flase positives but
increases the computation time.

Bloom filters are more effective when the number of bits produced by a hash
function is higher such as m = cn where c is a constant such as 5.

How do we get a family of k hash function?

We can use a basic MD5 or SHA hash function.

SHA(x), SHA(x  + i), or SHA (x||i) all work.


Example:
We insert and query on a Bloom filter of size m = 10 and number
of hash functions k = 3.

Let H(x) denote the result of the three hash functions which we
will write as a set of three values {h1(x), h2(x), h3(x)}

We start with an empty 10-bit long array:
	0 1 2 3 4 5 6 7 8 9
	0 0 0 0 0 0 0 0 0 0

Insert x0: lets say that generates the following bits.

H(x0) =  {1, 4, 9}
	0 1 2 3 4 5 6 7 8 9
	0 1 0 0 1 0 0 0 0 1

Next, insert x1:
H(x1) = {4, 5, 8}
	0 1 2 3 4 5 6 7 8 9
	0 1 0 0 1 1 0 0 1 1

Query y0:
H(y0) = {0, 4, 8} −→ No

Query y1:
H(y1) = {1, 5, 8} −→ Yes (False Positive)


We can minimize false positives when k = ln(m/n)
or (1/2)^k


Applications:

a) Preventing weak password choices: store dictionary of easily guessable
passwords as bloom filters. Query when users pick the passwords. Can add
previously used passwords in this list.


Note that deletion of values is not supported by a hash such as mentioned in
the above example.
In order support deletions, we need to have counting bloom filters.
Counting bloom filters hold a counter instead of a bit as a result of the
hash function.
However, the counting filters can undergo overflow.
Care needs to be taken that this does not happen.
In practise, a counting bloom filter takes 4 bits per position, it thus takes
upto 4 times more space than a standard bloom filter.


Hierarchical bloom filters:

We get additional checks to limit false positives.

Applications:
Deduplication: Block digest to find out if a block is present on the disk or
not. In case of a positive outcome - verify that a block indeed is present.
A negative outcome - add a digest to the deduplication system.


