# libketama - a consistent hashing algo for memcache clients #

We wrote ketama to replace how our memcached clients mapped keys to servers. Previously, clients mapped keys->servers like this:

```
server = serverlist[hash(key)%serverlist.length];
```

This meant that whenever we added or removed servers from the pool, everything hashed to different servers, which effectively wiped the entire cache. We add (and sometimes remove) servers from the memcached pool often enough to warrant writing this - if your memcached pool never changes, you can probably stop reading now :)

Ketama is an implementation of a consistent hashing algorithm, meaning you can add or remove servers from the memcached pool without causing a complete remap of all keys.

Here's how it works:

* Take your list of servers (eg: 1.2.3.4:11211, 5.6.7.8:11211, 9.8.7.6:11211)
* Hash each server string to several (100-200) unsigned ints
* Conceptually, these numbers are placed on a circle called the continuum. (imagine a clock face that goes from 0 to 2^32)
* Each number links to the server it was hashed from, so servers appear at several points on the continuum, by each of the numbers they hashed to.
* To map a key->server, hash your key to a single unsigned int, and find the next biggest number on the continuum. The server linked to that number is the correct server for that key.
* If you hash your key to a value near 2^32 and there are no points on the continuum greater than your hash, return the first server in the continuum.

If you then add or remove a server from the list, only a small proportion of keys end up mapping to different servers.


The majority of the code is a C library (libketama) and a php4 extension that wraps it. I've also included a class from our Java client. (Java Collections makes it rather easy). We use a single-server memcache client wrapped with a native php class to make it multi-server capable, so we just replaced the hashing method with a ketama_find_server call. (should be easy enough to plug this into libmemcache if need be)

http://static.last.fm/rj/ketama.tar.bz2
http://static.last.fm/ketama/ketama-0.1.1.tar.bz2
svn://svn.audioscrobbler.net/misc/ketama/

We've been using this in production for all our php installs and java services at Last.fm for around 10 days now. We deployed it just in time to smooth over moving loads of webservers between datacenters.