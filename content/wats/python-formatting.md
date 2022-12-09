---
title: "Python Formatting"
date: 2021-12-01T09:05:32-05:00
tags:
  - python
  - wat
summary: |
  ```python
  >>> a = {'a': {'b': 4}}

  >>> '{a["a"]["b"]}'.format(a=asdf)
  *** KeyError: '"a"'

  >>>'{a[a][b]}'.format(a=asdf)
  '4'
  ```
---

```python
>>> a = {'a': {'b': 4}}

>>> '{a["a"]["b"]}'.format(a=asdf)
*** KeyError: '"a"'

>>>'{a[a][b]}'.format(a=asdf)
'4'
```

Wat.

It turns out this is documented briefly in [PEP-498](https://peps.python.org/pep-0498/#differences-between-f-string-and-str-format-expressions),
although apparently not at all in the [official docs](https://docs.python.org/3/library/string.html#format-string-syntax).

> There is one small difference between the limited expressions allowed in str.format()
> and the full expressions allowed inside f-strings. The difference is in how index
> lookups are performed. In str.format(), index values that do not look like numbers
> are converted to strings

Essentially it boils down to the fact that fstrings take advantage of python's actual
parser, and therefore have syntax consistent with the rest of the language, whereas
`str.format` is approximating regular python syntax by reimplementing some common
access patterns in a mini lookalike-dsl.

And indeed if you attempt the same thing with an fstring:

```python
>>> f'{a["a"]["b"]}'
*** '4'
```

While some people may look at that and find the behavior unsurprising, I personally
think it looks like an unfortunate footgun that one would be forgiven to think
had different semantics than it has.

I can see what they were probably thinking at the time:

- There's going to be some additional complexity in parsing the extra syntax to support `a["a"]`
- What would `a[b]` even mean? Presumably `b` would have to be some additional `str.format`
  argument? (It gets a bit weird because it's not capturing local variables like fstrings do).
- Given the above, why force the user to type quotes all the time?

My hot take is that it's fine for `str.format` to not attempt to support arbitrary python syntax,
but that what it **does** support should have identical semantics. I.e. I should be able to copy
that portion of the format string out of the string and it should behave the same.
