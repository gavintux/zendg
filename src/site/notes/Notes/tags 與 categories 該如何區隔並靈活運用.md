---
{"created":"2025-11-03T10:02:13.671+08:00","aliases":null,"status":"#🌱","sourceType":"#📥/📖","author":["Perplexity AI"],"topics":["知識管理"],"categories":["Thinking"],"tags":["references"],"dg-publish":true,"permalink":"/notes/tags-categories/","dgPassFrontmatter":true,"updated":"2025-12-03T09:36:27.520+08:00"}
---

**tags（標籤）** 與 **categories（分類）** 在 Obsidian 中雖然都是整理筆記的工具，但目的與用法各有不同。以下幫你清楚區分，並給實用建議：

---

## 1. 概念區分

|功能|categories（分類）|tags（標籤）|
|---|---|---|
|用途|主題、範疇、內容性分類|屬性、狀態、橫向標記|
|表示方式|通常設在 YAML metadata區塊|可放 metadata，也可正文|
|結構|概念性分群、常用單一主題|扁平、可多重並存、多元|
|查找|聚合同分類所有筆記|快速篩選同屬性、狀態等|
|腳色|主要的知識組織骨幹|補充、橫向串聯、行動提示|

---
## 2. 實用舉例
**categories**：  
假如你有以下筆記：
- 校務會議記錄、年度計畫、教學反思、健康日誌等  
    你可用
- categories: [工作, 建議, 個人, 教學]
**tags**：  
針對筆記目前的狀態、屬性、或「有某種用途」的標記
- tags: [todo, meeting, important, draft, reference, idea]
---

## 3. 靈活運用原則

- **筆記主題、內容大範疇 → 放在 categories（如：工作、個人、教學）**
- **具體狀態、行動提示、橫向屬性 → 放在 tags（如：todo、important、meeting、已完成）**
- **一篇筆記可同時有多個 categories 和 tags，互不衝突，並可交叉查找**

---
## 4. 整合範例
```
---
 categories: [教學, 反思] 
 tags: [important, idea, draft] 
--- 
 # 教學反思—如何提升學生參與度`
```

這裡
- categories 標示這筆記屬於「教學」和「反思」範疇
- tags 標註這是重要的、帶有想法、還在草稿階段

---

## 5. 查找與管理

- 透過 Dataview 或搜尋功能，
    - 查詢 categories，可聚合同一主題的所有內容
    - 查詢 tags，可以跨不同主題，找出所有重要、待辦、會議等狀態筆記

---

**結論**：  
用 categories 建立你的筆記「樹狀分類骨幹」，用 tags 靈活標記屬性、狀態和橫向關聯，兩者搭配可做到知識組織與快速行動查找兼顧，形成彈性又高效的 Obsidian 筆記管理模式。