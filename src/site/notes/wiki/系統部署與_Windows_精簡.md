---
{"categories":"wiki","tags":["wiki","wiki/devops","wiki/sysadmin"],"status":"🌱","sourceType":"📥/✨","created":"2026-07-15T19:59:40.380+08:00","sources":["[[Clippings/GitHub 熱門開源神器！Winhance 一鍵安全精簡 Windows 11 臃腫預裝軟體與後台追蹤.md]]","[[Clippings/How to install Coolify Cloud on a server in Clouding.md]]"],"dg-publish":true,"permalink":"/wiki/系統部署與_Windows_精簡/","dgPassFrontmatter":true,"noteIcon":"","updated":"2026-11-26T16:18:11.085+08:00","dg-note-properties":{"categories":"wiki","tags":["wiki","wiki/devops","wiki/sysadmin"],"status":"🌱","sourceType":"📥/✨","created":"2026-07-15","sources":["[[Clippings/GitHub 熱門開源神器！Winhance 一鍵安全精簡 Windows 11 臃腫預裝軟體與後台追蹤]]","[[Clippings/How to install Coolify Cloud on a server in Clouding]]"]}}
---


# 系統部署與 Windows 精簡

本頁面記錄本機操作系統（Windows 11）精簡優化，以及 Linux 伺服器端私有雲應用部署（Coolify）的實踐筆記。

## 1. Windows 11 安全精簡神器：Winhance
Winhance 是 GitHub 熱門的開源系統優化工具：
* **主要功能**：
  * **一鍵精簡**：移除 Windows 11 內置的臃腫預裝軟體（Bloatware）。
  * **隱私防護**：關閉微軟後台的各類追蹤與遙測數據（Telemetry），釋放系統與網速頻寬。
  * **效能提升**：停用非必要的後台服務與開機啟動項，使系統運行更為流暢。

## 2. 伺服器端部署：Coolify 雲端安裝指南
Coolify 是一個強大的開源、自我託管的 Heroku / Netlify 替代方案：
* **安裝步驟**：
  1. 準備一台乾淨的 Linux 伺服器（建議 Ubuntu 22.04+）。
  2. 執行官方一鍵安裝腳本。
  3. 配置 Docker 守護進程。
* **優勢**：在單一 VPS 伺服器上，透過極簡的 Web UI 界面，便可輕鬆管理、部署與自動更新多個應用與資料庫（如 Node.js, Python, PostgreSQL 等）。

---

## 相關頁面
* [[wiki/Obsidian_高效工作流\|Obsidian_高效工作流]]
