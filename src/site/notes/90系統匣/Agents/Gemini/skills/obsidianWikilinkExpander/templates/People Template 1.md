---
{"categories":["[[People]]"],"tags":["people"],"birthday":null,"org":[],"created":"2026-06-21T20:16:52.465+08:00","dg-publish":true,"permalink":"/90系統匣/Agents/Gemini/skills/obsidianWikilinkExpander/templates/People Template 1/","dgPassFrontmatter":true,"noteIcon":"","updated":"2026-06-25T11:41:05.616+08:00","dg-note-properties":{"categories":["[[People]]"],"tags":["people"],"birthday":null,"org":[],"created":{"{ date }":null}}}
---

## Meetings


```base
filters:
  and:
    - categories.contains(link("Meetings"))
    - '!file.name.contains("Template")'
properties:
  note.date:
    displayName: Date
  note.people:
    displayName: People
  note.type:
    displayName: Type
  file.name:
    displayName: Meeting
  note.org:
    displayName: Org
views:
  - type: table
    name: Meetings
    order:
      - file.name
      - date
      - type
      - people
    sort:
      - property: date
        direction: ASC
  - type: table
    name: Person
    filters:
      and:
        - list(people).contains(this)
    order:
      - file.name
      - date
      - type
      - people
    sort:
      - property: date
        direction: ASC
  - type: table
    name: Type
    filters:
      and:
        - list(type).contains(this)
    order:
      - file.name
      - date
      - people
      - org
    sort:
      - property: date
        direction: DESC

```
