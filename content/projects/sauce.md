---
title: Sauce
template: projects
link: https://github.com/dancardin/sauce
date: 2021-01-23
weight: 3
---

A tool to help manage context/project specific shell-things like environment variables.

```bash
# Suppose you've got some directory structure
❯ mkdir -p projects/foo
❯ cd projects

# In "projects", you want some shorthand for quickly pushing your branches
❯ sauce new
Created /Users/danc/.local/share/sauce/projects.toml

❯ sauce set alias push='git push origin "$(git rev-parse --abbrev-ref HEAD)"'
Setting push = git push origin "$(git rev-parse --abbrev-ref HEAD)"

# Your project is, naturally, using 12-factor methodology, so you've got some
# project specific environment variables you need to load!
❯ cd foo
❯ sauce set env foo=bar AWS_PROFILE=meow
Setting foo = bar
Setting AWS_PROFILE = meow

# The core purpose!
❯ sauce
Sourced ~/.local/share/sauce/projects/foo.toml

❯ env
...
AWS_PROFILE=meow
foo=bar
```
