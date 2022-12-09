---
title: SQLAlchemy Model Factory
template: projects
link: https://github.com/dancardin/sqlalchemy-model-factory
weight: 4
---

A python library which makes it easy to write factory functions for sqlalchemy models, particularly for use in testing.

The most obvious comparison is against the much more popular python library, FactoryBoy.
By comparison, SQLAlchemy Model Factory removes a lot of the magic and machinery
introduced by factory boy.

One essentially writes regular functions, (which are directly callable as functions!),
which when called through the "ModelFactory" (`mf` by default) fixture, automatically
produces those objects in the database, similarly to FactoryBoy.

```python
# tests/test_example_which_uses_pytest
from sqlalchemy_model_factory import register_at
from . import models

@register_at('widget')
def new_widget(name, weight, color, size, **etc):
    """My goal is to allow you to specify *all* the options a widget might require.
    """
    return Widget(name, weight, color, size, **etc)

@register_at('widget', name='default')
def new_default_widget():
    """My goal is to give you a widget with as little input as possible.
    """
    return new_widget(name='default_name', weight=1, color='rgb(1, 2, 3)', size=4)


def test_example_model(mf, session):
    widget1 = mf.widget.new('name', 1, 'rgb(0, 0, 0)', 1)
    widget2 = mf.widget.default()

    widgets = session.query(Widget).all()
    assert len(widgets) == 2
    assert widgets[0].name == 'name'
    assert widgets[1].id == widget2.id
```
