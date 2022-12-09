---
title: SQLAlchemy Declarative Extensions
template: projects
link: https://github.com/dancardin/sqlalchemy-declarative-extensions
date: 2022-12-01
weight: 1
---

A python library for adding extensions to SqlAlchemy (and/or Alembic) which
allows declaratively stating the existence of additional kinds of objects
about your database not natively supported by SqlAlchemy/Alembic.

```python
@declarative_database
@as_declarative
class Base:
    schemas = Schemas().are("example")
    roles = Roles(ignore_unspecified=True).are(
        Role("read", login=False),
        Role(
            "app",
            in_roles=['read']
        ),
    )
    grants = Grants().are(
        DefaultGrant.on_tables_in_schema("public", 'example').grant("select", to="read"),
        DefaultGrant.on_tables_in_schema("public").grant("insert", "update", "delete", to="write"),
        DefaultGrant.on_sequences_in_schema("public").grant("usage", to="write"),
    )
    rows = Rows().are(
        Row('foo', id=1),
    )
    views = Views().are(View("low_foo", "select * from foo where i < 10"))


class Foo(Base):
    __tablename__ = 'foo'

    id = Column(types.Integer(), primary_key=True)


@view()
class HighFoo:
    __tablename__ = "high_foo"
    __view__ = select(Foo.__table__).where(Foo.__table__.c.id >= 10)
```
