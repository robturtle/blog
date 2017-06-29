---
title: Pitfalls in Python
date: 2017-06-29 03:26:02
tags:
- python
---

## objects as default parameter values
Here we use the term `object` to distinguish the stateful objects (lists, dicts) from immutable data (int). 
```python
def appendOne(lst=[]):
    lst.append(1)
    return lst
    
appendOne() # -> [1]
appendOne() # -> [1, 1]
```
Why: The right value is resolved once and been stored. The definition above is equivalent to:
<!-- more -->

```python
_defval = []

def appendOne(lst=_defval):
    lst.append(1)
    return lst
```
## tuple is not actually immutable
```python
t = ([],)
t[0].append(1) # acceptable!
```
Why: I consider it a bug of the CPython implementation. The implementation only reject the reassignment of the references the tuple holds. The mutation inside the referenced object is unprotected.

## the chain assignment is from left to right
```python
it = it.next = ListNode()
```
If this statement is writen in other languages, it's most likely to execute the assignment from right to left. It will first create a new object, assign it to the `next` attribue of the iterator, and move the iterator to this new instance.

But in python, it's resolved in opposite way. It will first assign the instance to the iterator, then make the `next` attribute refer to itself.

## trap in `list.__mul__`
```python
lst = [[]] * 3
lst[0].append(1)
print(lst) # -> [[1], [1], [1]]
```
Why: The expression in the list will be only resolved once. Because the `*` operator is nothing but a normal function call, the python interpretor will not treat it particularly. So the first statement is equivalent to:

```python
inner = [] # first resolve the inner object
tmp = [inner] # resolve outer object
lst = tmp.__mul__(3) # it will be resolved as [inner for _ in range(3)], i.e. [inner, inner, inner]
```

To construct multiple independent values in the list, we need to use list comprehension:
```python
lst = [[] for _ in range(3)] # here the inner expression "[]" will be resolved multiple times
lst[0].append(1)
print(lst) # -> [[1], [], []]
```
