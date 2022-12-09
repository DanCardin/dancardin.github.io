---
title: Configly
template: projects
link: https://github.com/schireson/configly
date: 2019-09-23
weight: 5
---

A python library to simplify and centralize the loading of configuration, such as environment variables.

```yaml
# config.yml
foo:
  bar: <% ENV[REQUIRED] %>
  baz: <% ENV[OPTIONAL, true] %>
list_of_stuff:
  - fun<% ENV[NICE, dament] %>al
  - fun<% ENV[AGH, er] %>al
```

```python
# app.py
config = Config.from_yaml('config.yml')

print(config.foo.bar)
print(config.foo['baz'])
for item in config.list_of_stuff:
    print(item)
```
