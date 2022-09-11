Against SLQ as a textual format
===============================

Why is there not a (C) library exposed by i.e. postgres which handles parsing locally for you?

It'd seem like there ought to be something which will parse SQL text and transform that into AST
Then further to take that AST and verify it syntactically

Then further to accept (the equivalent of) DDL and verify that the SQL is valid against that
schema.

Why does this not exist?

SQL is as a textual format is bad in numerous ways (trailing comma, etc)

But also if you're using some form of query builder (which, ignoring ORMs for a second)
(which seems objectively better than a random arbitrary string which you can only verify at
runtime, and is likely the source of sql injection...) then it's **got** all that information
that it then tosses in the trash in order to convert your SQL into a string.

Wouldn't it be better if query-builders/ORMs serialized straight into some AST-like format
and communicated that directly? (after locally verifying it against the aforementioned libpostgres.parser?)
