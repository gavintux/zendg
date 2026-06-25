---
{"categories":["[[Books]]"],"author":["[{{author}}]"],"cover":"{{coverUrl}}","genre":[],"pages":null,"isbn":"{{isbn10}} {{isbn13}}","isbn13":null,"year":null,"rating":null,"topics":[],"created":"2026-06-21T20:15:50.080+08:00","last":null,"via":"","tags":["books","references","to-read"],"dg-publish":true,"permalink":"/90系統匣/Agents/Gemini/skills/obsidianWikilinkExpander/templates/Book Template1/","dgPassFrontmatter":true,"noteIcon":"","updated":"2026-06-25T11:40:48.531+08:00","dg-note-properties":{"categories":["[[Books]]"],"author":["[{{author}}]"],"cover":"{{coverUrl}}","genre":[],"pages":null,"isbn":"{{isbn10}} {{isbn13}}","isbn13":null,"year":null,"rating":null,"topics":[],"created":"{{date:YYYY-MM-DD}}","last":null,"via":"","tags":["books","references","to-read"]}}
---


<%* if (tp.frontmatter.cover && tp.frontmatter.cover.trim() !== "") { tR += `![cover|150](${tp.frontmatter.cover})` } %>

<%* if (tp.frontmatter.localCover && tp.frontmatter.localCover.trim() !== "") { tR += `![[${tp.frontmatter.localCover}|150]]` } %>

![[{{title}}-{{author}}.jpg\|100]]
# {{title}}