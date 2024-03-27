---
title: Chapter Sync
template: projects
link: https://github.com/DanCardin/chapter-sync
date: 2024-02-27
weight: 2
---

A tool for recording serialized web content, in partial web-serial type novels, or other serialized content for which you want to be sent updates as they're published.

The other tools in the space (at least that I'm aware of: Leech, FanFicFare) are mostly designed around manual one-off usage of the tool to capture the current state of a story/series and turn it into an ebook.

chapter-sync, by contrast, records everything it captures to a sqlite database and will only collect new/missing chapters. It also bakes in (through supported subscription methods) the ability to send new chapters to "subscribers" who have not yet received it.
