---
tags:
  - Python/Dict
type: bookmark
source: https://pywheel.com/python-dict-under-the-hood/
---
---



Starting from this post, we begin to explore how different data structures and Python’s concepts work under the hood. In this series we will try to cover the most important implementation details and apply these knowledges in some real-world examples. A decent part of code snippets will be provided in C language. If you are not familiar with it, it’s a good reason to kickstart this language!

First, let’s define what dictionary is. Python dictionary is **an ordered\* hash table with open addressing** **using pseudo-random probing**.

> From Python 3.7 dictionaries are ordered.

“Ordered” means, that it maintains the order of insertion of key-value pairs. When iterating over, the items will be returned in the same order in which they were added. Open addressing with pseudo-random probing is one of implementations of collision resolution method. Throughout the article we will see how it works, but first, let’s recall some basics: what are the hash tables.

#### Hash tables

Hash tables are a fundamental data structure in computer science used for efficient data storage and retrieval. They work by mapping keys to values using a hash function. The hash value is used as an index to store and retrieve the associated value.

![Simple hash function diagram](https://i0.wp.com/pywheel.com/wp-content/uploads/2024/02/Untitled-Diagram-9-Page-2-2.png?resize=2372%2C508&ssl=1)

Example of simple hash table

Choosing a good hash function is crucial to minimize collisions and maintain the performance of the hash table. Here are the key characteristics, that hash function should have:

1.  Uniform distribution – distribute keys as evenly as possible across the range of hash values to minimize collisions.
2.  Deterministic – produce the same hash value for the same input.
3.  Fast – minimize the time required to compute the hash value, especially when dealing with large datasets.
4.  Secure – resist to various attacks, such as collision attacks and pre-image attacks.

For each type of object CPython uses different hash function. For example, to calculate the hash of integer number, it just performs modulus operation:

```
>>> import sys
>>> hash\_integer = 400 % sys.hash\_info.modulus
400
>>> hash(400)
400
>>> hash(-400)
-400
```


Where modulus equals to 2\*\*61 – 1 or 2\*\*31 – 1 for 64-bit and 32-bit platforms respectively. These numbers are called [Mersenne primes](https://en.wikipedia.org/wiki/Mersenne_prime). Mersenne prime is a prime number that is one less than a power of two. This is a common requirement for a good hash function to have modulus as a prime number. They offer good distribution and performance characteristics. However, here we can come up with a sequence of integers, that have the same hash value:

```
>>> number = 2\*\*61 - 1
>>> hash(number)
0
>>> hash(number \* 2)
0
>>> hash(number \* 200)
0
```

Doesn’t look good, right? Although it’s possible to “break” the hash function, on practice you will never face this situation, unless the developers are not in a sane mind.

> What do you think hash(-2) and hash(-1) will return? They will both return -2! This was made on purpose, because C language doesn’t have mechanism to handle exceptions, and -1 is used to indicate an error while hashing the key. So it just returns -2 instead ([pyhash.c#L185](https://github.com/python/cpython/blob/ecf16ee50e42f979624e55fa343a8522942db2e7/Python/pyhash.c#L185)).

For rational number the function may look a bit more complex, see [stdtypes.rst#L763](https://github.com/python/cpython/blob/aa8c1a0d1626b5965c2f3a1d5af15acbb110eac1/Doc/library/stdtypes.rst#L763):

```
import sys

def hash\_fraction(m, n):
   P = sys.hash\_info.modulus
   while m % P == n % P == 0:
       m, n = m // P, n // P

   if n % P == 0:
       hash\_value = sys.hash\_info.inf
   else:
       hash\_value = (abs(m) % P) \* pow(n, P - 2, P) % P
   if m < 0:
       hash\_value = -hash\_value
   if hash\_value == \-1:
       hash\_value = \-2
   return hash\_value
```

The main advantages of hash tables include fast average-case performance for insertion, deletion, and lookup operations, typically with constant time complexity O(1). However, their performance can degrade in certain scenarios, such as when there are many hash collisions or when the hash function has poor distribution. To mitigate collisions, hash tables often use techniques like **open addressing** or **chaining**. We will stop on the first one.

#### Open addressing

Open addressing is a technique used to resolve collisions that occur when two different keys hash to the same index. The main idea is to probe through the hash table, looking for an empty slot to place the collided element. There are different probing strategies, including linear probing, quadratic probing or double hashing, but we will take a look at **pseudo-random probing**.

```
find\_empty\_slot(PyDictKeysObject \*keys, Py\_hash\_t hash)
{
    assert(keys != NULL);

    const size\_t mask = DK\_MASK(keys);
    size\_t i = hash & mask;
    Py\_ssize\_t ix = dictkeys\_get\_index(keys, i);
    for (size\_t perturb = hash; is\_unusable\_slot(ix);) {
        perturb >>= PERTURB\_SHIFT;
        i = (i\*5 + perturb + 1) & mask;
        ix = dictkeys\_get\_index(keys, i);
    }
    return i;
}

```
The initial index `i` is computed by performing a bitwise AND operation between the hash value of the key and the mask, where mask is the number of slots in a hash table. If the slot is already occupied, we enter a probing loop. On each iteration, the index is re-calculated with the help of perturbation value. Perturbation value plays an important role in the probing algorithm. By initializing it `perturb = hash` and right shifting `perturb >>= PERTURB_SHIFT` on each iteration , it makes the probe sequence depends on every bit of the hash code. This solves the problems, which linear and quadratic probing algorithms are suffering from, called **primary** and **secondary clustering**.

![Primary clustering problem, python dictionary under the hood](https://i0.wp.com/pywheel.com/wp-content/uploads/2024/04/Untitled-Diagram-9-3-3-Page-6.png?resize=1024%2C200&ssl=1)

Primary clustering — elements cluster together, reducing the performance

![Secondary clustering problem, python dictionary under the hood](https://i0.wp.com/pywheel.com/wp-content/uploads/2024/04/Untitled-Diagram-9-3-3-Page-5.png?resize=1024%2C200&ssl=1)

Secondary clustering — long probe sequence

![Pseudo-random probing, python dictionary under the hood](https://i0.wp.com/pywheel.com/wp-content/uploads/2024/04/Untitled-Diagram-9-3-3-Page-4.png?resize=1024%2C200&ssl=1)

Pseudo-random probing

It’s important to note, that CPython uses `find_empty_slot` function not only to resolve collisions, but also to insert any new key. That means, that when we add a new key-value pair to a dictionary, we enter this probing loop.

==To delete a key from the hash table, we can’t just delete the slot, because it will break the subsequent probes. Instead, CPython marks the key as “dummy”== ([dictobject.c#L2484](#L2484)):

```
delitem\_common(PyDictObject \*mp, Py\_hash\_t hash, Py\_ssize\_t ix,
               PyObject \*old\_value, uint64\_t new\_version)
{
    ...
    dictkeys\_set\_index(mp->ma\_keys, hashpos, DKIX\_DUMMY);
    ...
}
```

The elements marked as “dummy” are cleaned up during resizing. CPython allocates a new table and reinserting all items again, simply skipping “dummy” elements. Thus, the new table may actually be smaller than the old one:

```
\>>> import sys
\>>> dct = {x: x\*2 for x in range(100)}
\>>> sys.getsizeof(dct)
4696
\>>> \[dct.pop(x) for x in range(30)\] \# remove the first 30 elements
\>>> sys.getsizeof(dct)
4696
\>>> dct\_new = dict(dct)
\>>> sys.getsizeof(dct\_new)
2272
```

#### Load factor

A load factor is a critical statistic of a hash table. It represents the ratio of the number of elements stored in the hash table to the total number of slots available. To maintain good performance this value should never be exceeded. With open addressing the acceptable value of load factor ranges between 0.6 and 0.75. When the load factor reaches its threshold, the hash table should be resized. CPython keeps this value under 2/3 and defines it as a macro ([dictobject.c#L539](https://github.com/python/cpython/blob/9578288a3e5a7f42d1f3bec139c0c85b87775c90/Objects/dictobject.c#L539)):
```
#define USABLE\_FRACTION(n) (((n) << 1)/3)
```

The initial size of an empty dictionary in Python is 8 ([dictobject.c#L111](https://github.com/python/cpython/blob/812245ecce2d8344c3748228047bab456816180a/Objects/dictobject.c#L111)). It allows the dictionary to store up to 5 elements without needing to resize (as 5 is less than 2/3 of 8). Once the number of elements exceeds this threshold, the dictionary will grow in size according to a predefined growth rate ([dictobject.c#L586](https://github.com/python/cpython/blob/9578288a3e5a7f42d1f3bec139c0c85b87775c90/Objects/dictobject.c#L586)):

```
#define GROWTH\_RATE(d) ((d)->ma\_used\*3)
```

Where `ma_used` is the number of items in the dictionary. So, that means, if we add 6th element to a dictionary, the new size will be 5 \* 3 = 15.

#### Python dictionary

##### Overview

To add a new key-value pair to a dictionary, the key object must be **hashable**. “Hashable” means, that it must implement two magic methods:

*   \_\_hash\_\_ — return hash value (integer) of an object;
*   \_\_eq\_\_ — compare two object for equality. This method is called with equals \== operator.

Overriding these methods for custom classes is optional. By default, they inherit from the base class “object”, which already defines these magic methods. However, if any of these methods is not implemented in a custom class, instances of that class cannot be used as keys in a dictionary:

class Fruit:
    def \_\_hash\_\_(self):
        raise NotImplementedError()

apple = Fruit()
fruits = {apple: "green"}  \# NotImplementedError is raised

class Fruit:
    def \_\_eq\_\_(self, other):
        raise NotImplementedError()

apple = Fruit()
fruits = {apple: "green"}  \# TypeError: unhashable type: 'Fruit'

Initializing and populating a dictionary with values can be done using a for-loop, passing an iterable, or providing key-value pairs. When the size of the dictionary is known beforehand, using an iterable or key-value pairs is preferable over a for-loop from the perspective of memory utilization. This is because CPython adjusts the size of the dictionary only once when initialized with an iterable or key-value pairs, rather than potentially resizing it multiple times within a for-loop. However, if such construction reduces readability of the code, then it’s better to use less efficient, but more clean way.


```
# For-loop
d = {}
for x in range(1000):
    d\[x\] = x\*\*2
# Dict-comprehension
d = {x: x\*\*2 for x in range(1000)}
# From iterable
d = dict((x, x\*\*2) for x in range(1000))
```

##### Time complexity

The table below shows the time complexity for the primary operations on a dictionary. It’s important to note that in worst-case scenarios, when hash collisions occur, the complexity can degrade to O(n), because CPython starts probing, which means iterating over the whole hash table.

![[attachments/Pasted image 20250116020452.png]]

##### Compact dictionary

Starting from version 3.6 python dictionaries were optimized to reduce the memory overhead. Here is an example of how dictionary was stored:

```
fruits = {"apple": 340, "mango": 800, "orange": 500}
entries = [
    ["--", "--", "--"\],  \# empty slot
    [-7729140759133891824, "orange", 500\],
    ["--", "--", "--"\],
    [-3224523775553202662, "apple", 340\],
    ["--", "--", "--"\],
    [-9139702805675118589, "mango", 800\],
    ["--", "--", "--"\],
    ["--", "--", "--"\],
]
```

Now it has two separate lists to store indices and entries:

```
indices = [None, 2, None, 0, None, 1, None, None\]
entries = [
    [-3224523775553202662, "apple", 340\],
    [-9139702805675118589, "mango", 800\],
    [-7729140759133891824, "orange", 500\],
]
```

In the previous layout, the total size of entries was calculated as 8 \* 24 = 192 bytes, with 8 entries each occupying 24 bytes. In the new layout, the size is reduced to (8 \* 1) + (3 \* 24) = 80 bytes. Small dictionaries have the most benefit, since pre-allocated empty slots only consume 1 byte in the indices list instead of the 24 bytes they previously occupied in the entries list. In addition to memory savings, the new layout also enhances iteration over keys, values, and items due to its denser structure.
