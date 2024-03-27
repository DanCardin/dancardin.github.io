---
title: Dataclass Settings
template: projects
link: https://github.com/DanCardin/dataclass-settings
date: 2023-10-29
weight: 2
---

A declarative settings library

dataclass-settings intends to work with any PEP-681-compliant dataclass-like object, including but not limited to:

* Pydantic models (v2+)
* dataclasses
* attrs classes

```python
from __future__ import annotations
from dataclass_settings import load_settings, Env, Secret
from pydantic import BaseModel


class Example(BaseModel):
    env: Annotated[str, Env("ENVIRONMENT")] = "local"
    dsn: Annotated[str, Env("DSN"), Secret('dsn')] = "dsn://"

    sub_config: SubConfig


class SubConfig(BaseModel):
    nested: Annotated[int, Env("NESTED")] = "4"


example: Example = load_settings(Example)

# or, if you want `nested` to be `SUB_CONFIG_NESTED`
example: Example = load_settings(Example, nested_delimiter='_')
```
