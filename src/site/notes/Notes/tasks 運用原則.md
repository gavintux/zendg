---
{"created":"2025-11-03T10:23:55.284+08:00","aliases":null,"status":"#🌱","sourceType":"#📥/📖","author":["Perplexity AI"],"topics":["知識管理"],"categories":["Thinking"],"tags":["references"],"dg-publish":true,"permalink":"/notes/tasks/","dgPassFrontmatter":true,"updated":"2025-12-03T09:37:08.261+08:00"}
---


「tasks」在 Obsidian 筆記系統中，通常作為標籤（tag）或屬性，用於管理行動項目、待辦事項、以及任何需要執行的工作，讓筆記系統具備待辦清單功能。以下是 tasks 具體運用方法及原則：

---

## 1. 什麼是 tasks 標籤？

**tasks** 就是給每個待處理、執行中的「任務」加註統一標記。所有有任務特性或行動要求的筆記，都加上 tasks tag，方便統一查找和追蹤。

---

## 2. 使用方法

- 在 YAML metadata 或正文中加入 tasks tag
    
    text
    
    `--- categories: [專案, 工作] tags: [tasks, urgent, todo] ---`
    
- 在筆記內容列出待辦項目
    
    text
    
    `- [ ] 完成校園設備檢查 - [ ] 回覆家長問題信件`
    
- 可搭配 Dataview 外掛查詢所有 tasks 標籤的筆記
    
    text
    
    `table file.mtime as "最近修改" from "" where contains(tags, "tasks") sort file.mtime desc`
    

---

## 3. 運用原則

1. **只給真有「行動項目」的筆記加 tasks**  
    避免泛用，讓 tasks 標籤只指向有待辦內容的筆記。
    
2. **統一命名，用複數形式 tasks（非 task）**  
    方便批次查找，避免混淆。
    
3. **搭配其它狀態標籤，如 todo、urgent、done**  
    能做更細致的分類（如 tasks+urgent 為緊急任務）。
    
4. **tasks tag 放在 YAML 區塊或正文，兩者可並用互補**  
    YAML 管理全局，正文可標記細項。
    
5. **定期查詢、整理 tasks 筆記**  
    避免任務遺漏，保持知識行動鏈有效流動。
    
6. **任務完成後可移除 tasks 標籤或加 done 標記**  
    讓系統持續保持簡潔。
    

---

**總結**：  
Obsidian 中的 tasks tag 是聚合與管理所有行動與待辦事項的最佳利器。只要有待處理任務，即加 tasks，並可搭配狀態標籤、查詢外掛，形成一個跨主題的動態行動管理系統！