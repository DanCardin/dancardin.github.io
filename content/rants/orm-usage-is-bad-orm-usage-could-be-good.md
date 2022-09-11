the canonical idea of a model is bad.

<insert> sqlalchemy model

it leads to impedance mismatch

<insert example here>

It would be better if we instead defined tables

<table>

and then explicitly defined mappers for those tables

<dataclass>

and our tooling then automatically allowed us to use those dataclasses as
though they were models, with the exception that now we can define models
unique to the situation in which we're querying for them

<query x with a one-to-many child y and get back Foo(children=[Bar(), ...])>

or 

<query x join y where you get a flat object which is the superset of the columns>

<reference to diesel, it's got good ideas but it's far too limiting>

<sqlalchemy though, can **look** a lot like sql, but still facilitate this usage>
