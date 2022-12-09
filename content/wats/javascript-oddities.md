---
title: "Javascript Syntax"
date: 2021-08-01T09:05:32-05:00
tags:
  - javascript
  - wat
summary: |
  ```javascript
  [1,2,3,4,].length // 4
  [1,2,3,4,,].length // 5

  27.toString() // Uncaught SyntaxError: Invalid or unexpected token
  27..toString() // '27'
  ```
---

```javascript
[1,2,3,4,].length // 4
[1,2,3,4,,].length // 5

27.toString() // Uncaught SyntaxError: Invalid or unexpected token
27..toString() // '27'
```

Wat.

In the former example, `[1,2,3,4,,]`, I simply dont understand the rationale.
Why is this not a syntax error!? 🤯

In the latter, `27.toString() // Uncaught SyntaxError: Invalid or unexpected token`. Okay fine,
not all languages support using attribute lookups on literal values like integers. Take python, for
example:

```python
>>> 1.__str__()
  File "<stdin>", line 1
    1.__str__()
      ^
SyntaxError: invalid syntax

>>> (1).__str__()
>>> x = 1
>>> x.__str__()

# Admittedly there's not a huge amount you can call on an int, necessarily. But still!
# Contrast it with something like `"".join`, and it starts to look weird!
```

And that's somewhat sad because other languages **do** support this kind of thing (like Rust),
and i think it ends up looking a lot more natural than the alternatives you get in languages
that dont support it.

But back to javascript `27..toString() // '27'`...wat again!?

Javascript has all sorts of weird **runtime** semantics that people meme about. The implicit
coersions, the insane equality map, etc. But things like these should literally just be
syntax errors, and I dont understand why they're not!
