title: Trivial caching with a simplified hash table
date: 2015-01-01
hidden: yes

[Hash tables](https://en.wikipedia.org/wiki/Hash_table) are pretty simple: [hash](https://en.wikipedia.org/wiki/Hash_function) your key to get an array index to look in for the value, and handle collisions... somehow. Most of the complexity comes from how you handle hash collisions. Now suppose that you handle them *by not handling them:* when you're trying to write to the hash table, just overwrite anything that's already there. This will give you a [cache](https://en.wikipedia.org/wiki/Cache_%28computing%29): you put things in, but they might disappear. Newer things will tend to replace older ones.

I love how simple this is. The usual way of making a cache in memory is to keep a doubly linked list of the items in the cache in order of how recently they were used, have a hash table pointing to them, and do a bunch of twiddling to make sure that when the cache fills up, the least recently used item is evicted first. That's fine, but it's got a fair bit of overhead, and it's not *trivial* the way the hash table thing is.

Let's write some code. I'll use C, the lingua franca of people whose concern for optimization is excessive but not quite obsessive. First let's define some data types:

```c
// Keys and values in the cache are both strings. You could use
// whatever you want, but let's keep it simple for now.
typedef char* key_t;
typedef char* value_t;
// A cache is just an array of slots where we can put values.
typedef struct {
    key_t key; 
    value_t value;
} cache_entry_t;
typedef cache_entry_t* cache_t;
```

Making a cache just requires allocating some memory to hold it, and making sure it's initially blank:

```c
#define CACHE_SIZE 128
cache_t cacheCreate(void) {
    return calloc(CACHE_SIZE, sizeof(cache_entry_t));
}
```

To insert a key-value pair into the cache, hash your key and put it in there.

```c
void cacheInsert(cache_t cache, key_t key, value_t value) {
    int i = hash(key) % CACHE_SIZE; // Get position in cache
    free(cache[i].key);             // Evict anything already there
    free(cache[i].value);
    cache[i].key = key;             // Put new value in place
    cache[i].value = value;
}
```

To look up something, it's just like a hash table lookup except without any collision handling:

```c
value_t cacheGet(cache_t cache, key_t key) {
    int i = hash(key) % CACHE_SIZE;
    if (cache[i].key && keys_equal(cache[i].key, key)) {
        return cache[i].value;  // We found the key! :-D
    } 
    return NULL;                // We didn't find the key. :-(
}
```

There you have it: an in-memory cache, with negligible overhead, in just a few lines of simple code. Fun, right?

(Want complete, runnable code? [Here's a repo.](https://github.com/dparrol/hashcache))
