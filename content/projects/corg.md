---
title: corg
template: projects
link: https://github.com/DanCardin/corg
date: 2022-01-15
weight: 10
---

A cog-like tool, written in Rust.

Straight from Ned:

> Cog is a file generation tool. It lets you use pieces of Python code as generators in your source files to generate whatever text you need.

Instead, Corg allows one to choose any executable (python, bash, etc) which accepts piped input. Shown below, Corg uses a shebang-looking mechanism instead.

The most obvious motivating example which comes to mind for this tool is for keeping documentation up-to-date with their sources of truth. Be that, verifying CLI help text, or executing code examples.

```markdown
<!-- [[[#!/usr/bin/env bash
cargo run --features cli -- --help
]]] -->
```
