---
tags:
  - Python/List
type: bookmark
source: https://pywheel.com/how-python-lists-work-under-the-hood/
---

> CPython operates with the array of references, which is guaranteed to be stored contiguously in memory. 
> Why does it matter? Well, in computer science there is a concept called [spatial locality](https://en.wikipedia.org/wiki/Locality_of_reference#:~:text=Spatial%20locality%20(also%20termed%20data,in%20a%20one%2Ddimensional%20array.). It is based on the principle that if a program accesses a particular memory location, it is likely to access nearby memory locations in the near future. Prefetching mechanisms in processors can predict and fetch data into the cache before it is actually needed based on spatial locality patterns. This principle is crucial for optimizing memory access and cache efficiency.---


![[attachments/Pasted image 20250118121323.png]]
> ==The “append” command is quite efficient, with a time complexity of O(1)==. Regardless of the size of the list, it is just a simple assignment operation with a constant time. However, if an ==item is inserted at an arbitrary position using list.insert(index, object), the time complexity degrades to O(n)==, because all the elements in the array of references after the insertion point need to be moved.

> Unlike dictionaries, lists in Python do not have a load factor, which indicates how full a collection of elements can be before having to resize it. Instead, when the length of a list reaches a threshold, it is getting resized according to a growth pattern.

> The ==“remove” command in Python has a time complexity of O(N)== for the same reason as the insertion command: all elements after the removed one need to be shifted. On the other hand, ==popping the last item of the list has a constant time complexity O(1), regardless of the size of the list.==

![[attachments/Pasted image 20250118122000.png]]

CPython treats (homogeneous or heterogeneous) similarly in most cases, using the same algorithms, except sorting. 
> For homogeneous lists CPython will simply invoke the ==key_richcompare== function specific to the object’s type
> If lists are heterogeneous lists, it calls slow and complex function **PyObject_RichCompareBool**, which performs rich comparison between two objects

