---
{"dg-publish":true,"permalink":"/Categories/Yi 知識庫/","dgPassFrontmatter":true,"noteIcon":"","created":"2025-11-02T22:01:13.009+08:00","updated":"2026-05-28T15:37:43.310+08:00","dg-note-properties":{}}
---



```base
filters:
  and:
    - file.tags.contains("yi")
    - '!file.name.containsAny("Template")'
views:
  - type: table
    name: 易經知識
    order:
      - file.name
      - categories
      - created
      - tags
    sort:
      - property: tags
        direction: ASC
      - property: tage
        direction: ASC
      - property: file.tags
        direction: ASC
    columnSize:
      file.name: 405
      note.created: 149

```
