---
title: Pytest Mock Resources
template: projects
link: https://github.com/schireson/pytest-mock-resources
weight: 2
date: 2019-01-09
---

A python library which produces pytest fixtures which let you test against
external resource (Postgres, Mongo, Redshift...) dependent code, by orchestrating
running real instances of those services.

```python
from pytest_mock_resources import create_postgres_fixture
from models import ModelBase

pg = create_postgres_fixture(ModelBase, session=True)

def test_view_function_empty_db(pg):
  response = view_function(pg)
  assert response == ...
```
