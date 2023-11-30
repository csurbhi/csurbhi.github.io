Memcached is a distributed dictionary; it has a client and many servers.
The client receives keys that it tries to locate, it calculates the hash and
finds out what server it is stored on. It then sends the request to that
server and then returns the object back. The client then aggregates all the
objects for the application. In a nutshell this is what it is.

If you're new to hashes, here's a quick overview. A hash table is implemented as an array of buckets. Each bucket (array element) contains a list of nodes, with each node containing [key, value]. This list later is searched to find the node containing the right key. Most hashes start small and dynamically resize over time as the lists of the buckets get too long.

In very simple language, memcached is a two level hash. The first level
indicates what server contains the final object. The second level hash runs on
the server and returns the actual object we are interested in it.

Each memcached object (a server) runs independently of the other. There is no
synchronization, no cross talk, no broadcasting, no replication.
At the bottom of it,
memcached is a cache that is implemented using a hash. A cache cannot contain
all the elements but only a few hot ones. Thus the hash has to evict some
objects that are deemed cooler. memcached uses a "Least recently used" cache
scheme, items expire after a minute to prevent stale data from being returned
or unused data is evicted to allow the entry of newer data. When data requires
to be invalidated, the database server is directly notified about this. 
Free space in the hash is lazily reclaimed. Queries do not stall for the
reclaimation process to complete.

Object cache on the server may be optionally
replicated to make sure that when one dies another is available. But
applications cannot rely on this as they should be able to work around a cache
miss. All the synchronization necessary for distributed databases to work is
already present in the database servers and memcached relies on this.

Memcached is written in C and uses a slab allocator to allocate memory.
It uses asynchronous i/o and a libevent library for I/O completion
notifications. On Linux, this library uses epoll as that works well with
millions of concurrent requests. Inside memcached all algorithms have O(1)
time complexity. Memcached is also lockless. All objects are mulitversioned
internally and are also reference counted. Thus no client can block another
clients object. If one client is updating an object stored in Memcached while a dozen others are downloading it, even with one client on a lossy network connection dropping half its packets, nobody has to wait for anybody else.

A final optimization worth noting is that the protocol allows fetching
multiple keys at once. Thus an application can request thousands of keys at
once instead of requesting them one at a time. This makes the request
completion faster as the servers can address the requests in parallel.
When necessary, the client libraries automatically split multi-key loads from the application into separate parallel multi-key loads to the Memcached instances.
Alternatively, applications can provide explicit hash values with keys to keep groups of data on the same instance. That also saves the client library a bit of CPU time by not needing to calculate hash values.

memcached is a cache but can also be used a in-memory database. This can be
used for warding off malicious bots. By keeping track of the last times and actions of each IP address and session, our code automatically can detect patterns and notify us of attacks early on, taking automatic action as necessary. Storing this information in the database would've been wasteful, burdening the disks unnecessarily. Putting it in memory is fine, however, because the data is safe to lose if a Memcached node fails.

The author of memcached "Brad Fitzpatrick" has written an article on it from where most of this
blogs contents is based on. He found out that many people (in the initial
days) used memcached for the following:
1) Many people use it like we do on LiveJournal, as a typical cache for small Web objects. 
2) One site is using it to pass the currently playing song from their Java streaming server to their PHP Web site. They used to use a database for this, but they report hitting Memcached is much nicer. 
3) A lot of people are caching authentication info and session keys. 
4) One person reported speeding up mail servers by caching known good and known bad hosts and authentication details. 

If your database runs on a single node, you probably do not need memcached.
You can use shared memory for this purpose.
memcached is useful when your database is itself distributed on different
nodes, as it offer distributed caching support to it.

