---
{"categories":"[[People]]","tags":["people","authors"],"dg-publish":true,"permalink":"/90系統匣/Agents/Gemini/skills/obsidianWikilinkExpander/templates/Author Template1/","dgPassFrontmatter":true,"noteIcon":"","created":"2026-06-21T20:17:25.231+08:00","updated":"2026-06-25T11:40:33.308+08:00","dg-note-properties":{"categories":"[[Categories/People]]","tags":["people","authors"]}}
---

## Books


```base
filters:
  and:
    - categories.contains(link("Books"))
    - '!file.name.contains("Template")'
properties:
  note.author:
    displayName: Author
  file.name:
    displayName: Name
  note.year:
    displayName: Year
  note.genre:
    displayName: Genre
views:
  - type: table
    name: Author
    filters:
      and:
        - list(author).contains(this)
    order:
      - file.name
      - year
      - genre
    sort:
      - property: genre
        direction: ASC

```
