---
title: "Python Slice Assignment"
date: 2022-12-01T09:05:32-05:00
tags:
  - python
  - wat
summary: |
  ```python
  >>> asdf = [1, 2, 3, 4]
  >>> asdf[0:0] = [-1, 0]
  >>> asdf
  [-1, 0, 1, 2, 3, 4]
  ```
---

```python
>>> asdf = [1, 2, 3, 4]
>>> asdf[0:0] = [-1, 0]
>>> asdf
[-1, 0, 1, 2, 3, 4]
```

Wat.

This little nugget is called "slice assignment" (so far as I know). It's mutating
the list, putting the contents of the right hand list into the left hand one.

Slice assignment is a little used feature, probably likely because python does
a pretty bad job of documenting a bunch of its syntactical features. I couldn't
find any PEPs referencing slice assignment, and there aren't any obvious
[references in the docs](https://docs.python.org/3/search.html?q=slice+assignment&check_keywords=yes&area=default)
either. If you search for "slice assignment", you definitely get a bunch of results,
just not from python itself.

It turns out that python lacks list methods or other options for efficiently
altering lists beyond appending. You get `[].append`, and `[].extend`. Then you
get `[].insert`, but no corresponding equivalent to `extend`. Alternatively,
you could just live with unpacking or adding the two lists together and allocating
a new one.

But it gets pretty funny. Python supports all sorts of extra slicing syntax options,
that can be pretty useful **normally**.

You want the last item in a list? `a[-1]`. Nice, concise, clear!

You want the even indices from a list? `a[::2]`. Concise sure, arguably less clear until
you know the synax, probably still ultimately nice.

Now use those on the assignent side!

```python
>> a = [1, 2, 3, 4, 5, 6, 7, 8, 9]
>>> a[::-2] = [-1, -2, -3, -4, -5]
>>> a
[-5, 2, -4, 4, -3, 6, -2, 8, -1]
```
