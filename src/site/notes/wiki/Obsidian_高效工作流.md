---
{"categories":"wiki","tags":["wiki","wiki/obsidian","wiki/ai"],"status":"🌞","sourceType":"📥/✨","created":"2026-07-15T19:59:38.352+08:00","updated":"2026-11-25T21:25:21.782+08:00","sources":["[[Clippings/How I use Obsidian.md]]","[[Clippings/Obs＃50 QuickAdd全攻略(一)：改變工作流程的超強外掛 – 簡睿隨筆.md]]","[[Clippings/【解答合集】批量添加删除修改查找替换 - 经验分享.md]]","[[Clippings/Mindomo API.md]]","[[Clippings/國外瘋傳的 AI 筆記工作流！LLM Wiki 讓你的 Obsidian 筆記升級成能自我進化的第二大腦！.md]]"],"dg-publish":true,"permalink":"/wiki/Obsidian_高效工作流/","dgPassFrontmatter":true,"noteIcon":"","dg-note-properties":{"categories":"wiki","tags":["wiki","wiki/obsidian","wiki/ai"],"status":"🌞","sourceType":"📥/✨","created":"2026-07-15","updated":"2026-09-24","sources":["[[Clippings/How I use Obsidian]]","[[Clippings/Obs＃50 QuickAdd全攻略(一)：改變工作流程的超強外掛 – 簡睿隨筆]]","[[Clippings/【解答合集】批量添加删除修改查找替换 - 经验分享]]","[[Clippings/Mindomo API]]","[[Clippings/國外瘋傳的 AI 筆記工作流！LLM Wiki 讓你的 Obsidian 筆記升級成能自我進化的第二大腦！]]"]}}
---


# Obsidian 高效工作流

本頁面彙整並解析如何運用 Obsidian 外掛、API 與批次處理技巧，打造無縫、流暢的個人生活操作系統（Life OS）。

## 1. QuickAdd 外掛全攻略
QuickAdd 是改變 Obsidian 工作流的靈魂外掛，主要包含四大核心功能：
* **Template (模板模式)**：一鍵調用指定模板在特定資料夾中生成筆記，並可自動填寫日期與 Frontmatter。
* **Capture (捕捉模式)**：無需打開筆記，直接將想法、任務或當下心情 append 到指定筆記（如日誌、Inbox）的特定位置。
* **Macro (巨集模式)**：組合多個指令與 Javascript 腳本，實現高度自動化工作流。
* **Multi (多重指令)**：將多個 Capture、Template 組合成一個下拉選單供快速點選。

## 2. 批量添加、刪除與修改 (正則與工具)
處理知識庫（Vault）中海量檔案時，有時需要全局修改格式：
* **正則表達式 (RegEx)**：利用 VS Code 等編輯器，使用 `^(.*)$` 等正則查找語法進行全 Vault 的批次取代。
* **Frontmatter 批次修改**：在全域替換 YAML 屬性（如增刪 `categories` 或 `tags`）時的格式替換範例。

## 3. 個人工作流實踐：How I use Obsidian
* **Inbox 緩衝區**：所有 Clippings 或雜亂想法先落地在一個緩衝區（如 `00inbox/`），定期進行歸納，避免大腦超載。
* **心智圖整合 (Mindomo API)**：透過 API 將 Obsidian 中的大綱結構與外部心智圖工具 Mindomo 連接，實現視覺化思考的快速切換。

## 4. AI 賦能的第二大腦：LLM-Wiki 自動化編譯工作流
傳統 Obsidian 工作流仰賴手動建立雙向連結與整理標籤，維護負擔龐大。導入 [[wiki/LLM_Wiki_知識庫架構\|LLM_Wiki_知識庫架構]] 與 AI Agent 後，Obsidian 升級為兼具視覺化介面與 AI 自主編譯的第二大腦：
* **Web Clipper 快速擷取**：透過瀏覽器擴充功能，將高品質網路文章一鍵剪藏至 `Clippings/` 或 `raw/` 原始資料庫。
* **AI 代理自動編譯 (Ingest)**：AI 讀取剪藏原始文件，提取概念與實體並寫入 `wiki/`，同時自動維護全庫索引 `index.md` 與時間軸日誌 `log.md`。
* **自動排程與維護 (Schedule & Linting)**：透過排程定時檢查新進資料，並由 Agent 自動進行死連結排查、孤立頁面診斷與內容矛盾警告，維持全庫極致整潔與高連通度。
* **視覺化延伸**：利用 Agent 將 Markdown 筆記一鍵轉為互動式 HTML 時間軸與資訊圖表，提升知識探索體驗。

---

## 相關頁面
* [[wiki/LLM_Wiki_知識庫架構\|LLM_Wiki_知識庫架構]]
* [[Notes/PAPAYA 電腦教室\|PAPAYA 電腦教室]]

Path: wiki/Obsidian_高效工作流.md
