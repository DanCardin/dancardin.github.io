---
title: Unit testing is overrated
date: 2025-11-05
summary: But automated testing in general, is not!
draft: true
---

Testing, in whatever form, is an important part of developing software.
Much debate has been had on the internet about what **exactly** is "unit" vs "system"
vs "integration" tests, and when to use which one and how much of each to write.
I'm not sure whether this is an unpopular opinion or not, but I think testing "units",
(as most people understand a "unit" to mean) is often pointless, counterproductive,
and unnecessarily limiting.

# Test the system
I think testing "units", as in individual functions, is useful only insofar as
the function under test is an external API to the system under test.

If you're authoring a library, it obviously makes sense to directly test the functions
you're exposing as part of the API. I think it's much less obvious that it's useful
to test almost any individual internal-only function (although I wouldn't definitively
say you **shouldnt**, it's certainly case-dependent). If I **can** test a behavior by
utilizing the public API, I almost always will, even if it's more challenging.

For a service like an API, I would greatly prefer exclusively testing the API endpoints
themselves, as opposed to testing the handler functions sans the API framework machinery.
There's simply too much that can be different between the test-only setup and the deployed
system.

I think you will almost universally get more bang for your testing buck by testing
at the exterior boundaries of your system versus individually testing functions that
are implementation details which might change more frequently.

It helps to ensure your tests are more flexible and only break when your code is
**actually** breaking exterior-facing backwards incompatibilty. 

# No really, test the **SYSTEM**
The thing which usually prevents actually testing the system, or at least makes it more
challenging is: IO.

Take some random (FastAPI) API endpoint.

```python
from sqlalchemy.orm import Session
from models import Thing

def create_a_thing(postgres: Session, name: str):
    thing = Thing(name=name)
    postgres.add(thing)
    postgres.commit()

# or perhaps, more directly

from sqlalchemy import tet

def create_a_thing(postgres: Session, name: str):
    query = text("INSERT INTO thing (name) VALUES (:name)")
    postgres.execute(query, {'name': name})
    postgres.commit()
```

How do you test this?

Anecdotally, many people decide that you shouldn't test things like SQL and want to
mock SQL calls. I would argue that for a function like this, that completely ruins
your test to the point that you might not as well write one.

Alternatively, I've heard that this implementation is bad. You should factor the code
with patterns that abstract the **actual** SQL execution somewhere else, so that your
business logic is tested and the actual "glue" that is the SQL execution is or is not
tested separately -- to which I would argue that I that's simply asking a lot, and is
almost certainly going to lead to less well tested code that's harder to follow.

Perhaps some others would reach to stub the postgres connection for a python implementation,
or even SQLite. Now we're moving in the right direction, however in both cases, there
is a very real possibility of drift in **behaviors** between postgres and anything else,
let alone that it constrains you to the most-common featureset of the postgres and the stub.

---

What I think is the **only** good solution is to spin up a **REAL** postgres and send
a connection to that into your function.

This is potentially admittedly challenging! You have to worry about shared test state
among tests, or you need to (slowly) stand up a separate database per test. This problem
at least is less a strategic difference so much as it is a lack of infrastructure.

At $job, for a long time we struggled with various alternatives: controlling the transactions
to avoid committing data that might interfere, writing data but carefully controlling the tests
such that they didn't *care* about what other tests were doing, etc.

In the end [pytest-mock-resources](https://pytest-mock-resources.readthedocs.org) was our solution.
For postgres, and for most of our applications, it will spin up a single postgres docker container,
then using Postgres' [template database](https://www.postgresql.org/docs/current/manage-ag-templatedbs.html)
feature, create a DDL-only template, and use that to near-instantly clone a new empty database
per test.

This for most usecases that blindly operate within a single database, this provides maximal
equivalence to the real system because it's a real postgres with the real DDL that can only
be affected by the currently running test.

# Avoid Mocks

The above is an example of how, wherever possible I think it's best to avoid Mocks. The function
under test **accepts** a postgres connection rather than creating one itself, which allows the test
infrastructure to produce the connection for the test.

As much as is practical to have **real** instances of IO-generating dependencies, I would always
prefer these to mocks. Need S3? [moto server](https://docs.getmoto.org/en/latest/docs/server_mode.html)
Moto the library **frequently** clashes with `boto3` or other libraries specifically **because**
it is internally mocking, which ultimately makes it fairly fragile. However moto server ensures that
the code under test (as far as its concerned) **is** talking to a real service in exactly the
same way it would in a deployed environment.

HTTP Requests potentially pose an interesting challenge. A common python library people use is 
[responses](https://github.com/getsentry/responses) or similar. But each of these **are** actually
ultimately mocking the underlying HTTP requests. This is certainly better than literally
`unittest.mock.Mock`ing the function calls away, but I've at least prototyped
[a library](https://github.com/DanCardin/responsaas) which spins up a real HTTP server with APIs
for controlling how it will respond within a given test.
