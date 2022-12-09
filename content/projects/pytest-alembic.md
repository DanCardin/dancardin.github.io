---
title: Pytest Alembic
template: projects
link: https://github.com/schireson/pytest-alembic
date: 2020-02-29
weight: 1
---

A Pytest plugin to test alembic migrations (with default tests) and which
enables you to write tests specific to your migrations.

```bash
$ pip install pytest-alembic
$ pytest --test-alembic

...
::pytest_alembic/tests/model_definitions_match_ddl <- . PASSED           [ 25%]
::pytest_alembic/tests/single_head_revision <- . PASSED                  [ 50%]
::pytest_alembic/tests/up_down_consistency <- . PASSED                   [ 75%]
::pytest_alembic/tests/upgrade <- . PASSED                               [100%]

============================== 4 passed in 2.32s ===============================
```
