The pigeon hole principle states that when you try to fit in n pigeons in m
holes where m < n, a few of the holes will have more than one pigeon. Hashing
is a technique of mapping a large dictionary to a small sized hash table.
Collision is inevitable in this case.
Collision can be prevented using Linear Probing or Open Addressing.

Chaining: In this we simply maintain a linked list and store the elements in
it. In the worst case, the entire linked list requires walking through to find
the key, this causes O(n) complexity as all the nodes could be attached to
this one linked list. This is however acceptable when the number of collisions
are far and few.

Linear Probing: You check the original location through hashing, if the key is
not present at this location you linearly look through the next locations. The
lookup can in the worst case take O(m) where m is the size of the lookup
table. Theoretically, the ith position tried after the initial one is k+i.
An obvious solution is to try the k + ai th position where a is coprime to the
&length of the table. The major problem that appears with linear search is that
if two sequences originating from k1 and k2  come together so that:
k1 + ai = k2 + aj
then all subsequent entries in one appear in the search of the other. Once the
table starts filling up, the sequences tend to join with other clusters. This
makes search much longer than expected.

Quadratic Probing: avoids clustering by defining the ith location as follows:
k + ai + bi^2
The next position in a sequence now depends on the length of the sequence, so
random collisions of two sequences do no coincide or combine.

In practise linear probing is observed to perform better than quadratic
probing, as linear probing improves CPU cache hits ratio! (the array is
sequentially cached in the CPU lines, so accessing the next location involves
looking at cached data).


(https://opendatastructures.org/newhtml/ods/latex/hashing.html#tex2htm-68)

Tabulative hashing:
When hashing produces values that are uniformly random, collisions can be
prevented on hot keys.
One way to achieving this is to store a giant array tab of length 2^w, where
each entry is a random w-bit integer, independent of all other entries.
Unfortunately 2^w is a prohibitive size, instead we look at a table size of
2^(w/r); we look at each integer as having a size of w/r.
Thus we split the w bit integer into multiple integers of w/r size and combine
their value by oring them.

The hash function can be implemented as follows when w is 32, r is 4 and w/r
is 8:
hash(x) = (hashCode(x) >> 8  & 0xff
		^ hashCode(x) >> 8  & 0xff
		^ hashCode(x) >> 16  & 0xff
		^ hashCode(x) >> 24  & 0xff) >> (w-w/r)

One can verify that the value generated is uniformly spread over [0, ...,
2^(w-w/r)]. Similarly, the hash values generated are independent of each
other.
hashCode() can be MD5, SHA1 etc.


Multiplicative hashing: Multiplicative hashing is an efficient method to
generate hash values based on modular arithmetic and integer division. It uses
the div operation that calculates the integral part of a quotient while
discarding the remainder.

In multiplicative hashing, we use a hash table of size 2^d for some integer d
(called the dimension). The formula for hashing an integer x  belongs to [0,
2^w -1] is
hash(x) = ((z . x) mod 2^w)/(2^(w-d))

Here z is a randomly chosen odd integer in the set [0, 2^w -1]
w is the number of bits in an integer.
Dividing by (2^(w-d)) is equivalent to dropping the rightmost w-d bits.
Thus the code can be reduced to:
hash(x) = hashcode(z * x) >>  (w-d)
hashCode could be MD5 or SHA1 or some function of the sort.
Multiplicative hashing does a good job of avoiding collisions.


It has been observed that multiplicative hashing works better than tabulative
hashing as slicing the hash values into bits is a slower order of job than
simply dropping out bits.
